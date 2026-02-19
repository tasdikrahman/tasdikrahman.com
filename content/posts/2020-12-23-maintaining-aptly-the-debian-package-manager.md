---
title: "Maintaing aptly - The debian package manager"
description: "Maintaing aptly - The debian package manager"
tags: [devops]
comments: true
share: true
cover_image: '/content/images/2020/12/aptly.png'
---

This post is a continuation of this [tweet thread](https://twitter.com/tasdikrahman/status/1326536874375090176)

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Sad to see aptly slowly <a href="https://t.co/zkXgsruAGi">https://t.co/zkXgsruAGi</a> rotting, but works really well till the last 1.4.0 build as a debian package repository for your needs. (1/n)</p>&mdash; Tasdik Rahman (@tasdikrahman) <a href="https://twitter.com/tasdikrahman/status/1326536874375090176?ref_src=twsrc%5Etfw">November 11, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

[Aptly](https://www.aptly.info/) is a debian package repository, the specific use case which we are using it for is pushing out application specific debian packages which will then be pulled out while deploying a new SHA/version of the application, to the app boxes. More on this in another post. But what this post will concentrate on, are a few things which we discovered while maintaining aptly, storing packages which ran into storage spaces consuming multiple TBs.

## Use an SSD

This is a must here, since aptly serves the debian packages straight from the filesystem. If you are not using an SSD, you will definitely see a slowup in the package fetch/insert steps.

## Add your cleanup scripts early

Make it known the application owners, that you will only keep, say last 10-15 packages in the filesystem. Given that the deb package would roughly take a few hundred MiBs(very rough estimate, can vary for you) you will reach a point where your initial storage will run out over time.

Throwing disk and compute can be done and is all fine, but the former will reach a limit, when your filesystem itself can't support it any further (ext4, supports logically uptil 1Exbibyte (1EiB), but the point is that you don't really end up in a situation like that where cleanup becomes a problem)

Even then, resize2fs (version 1.42.9) would simply fail if you tried increasing the disc more than 16TB, saying that the new size is too large to be expressed in 32bits. Adding to it, if you are running an older kernel version, eg 3.19.x which doesn't handle 64 bit ext4 filesystems properly, you would be left in a puddle here.

## Build time slows down over time

As with time, the index of the packages held inside aptly will grow, which will considerably increase the package publish step for your package consumers, which in most environments is a huge productivity kill. Imagine having to wait x amounts of minutes every now and then while trying to push a commit and deploying it over to the app boxes. If you combine this to the number of developers in the team/company, those are a lot of people hours right there.

The particular step is the package publish step, for a distribution which slows the whole process.

## How to prevent this API slowdown in the publish step?

The publish step [https://www.aptly.info/doc/aptly/publish/repo/](https://www.aptly.info/doc/aptly/publish/repo/) has an option/flag called `--skip-contents`, which will essentially not generate the index of the contents stored. We had tried unsuccessfully storing this in the aptly app config, but it seemed to not work.

After checking the [codebase](https://github.com/aptly-dev/aptly/blob/24a027194ea8818307083396edb76565f41acc92/api/publish.go#L232), in the specific route, which was `/publish/:prefix/:distribution` which used to take the most time, and for which we wanted to set the above setting.


```golang
// https://github.com/aptly-dev/aptly/blob/24a027194ea8818307083396edb76565f41acc92/api/publish.go#L232
// PUT /publish/:prefix/:distribution
func apiPublishUpdateSwitch(c *gin.Context) {
	param := parseEscapedPath(c.Params.ByName("prefix"))
	storage, prefix := deb.ParsePrefix(param)
	distribution := c.Params.ByName("distribution")

	var b struct {
		ForceOverwrite bool
		Signing        SigningOptions
		SkipContents   *bool
		SkipCleanup    *bool
		Snapshots      []struct {
			Component string `binding:"required"`
			Name      string `binding:"required"`
		}
		AcquireByHash *bool
	}

	if c.Bind(&b) != nil {
		return
	}
...
...
```

The var `b` would get it's de-serialised content for `SkipContents` is what we anticipated, when we started passing the value `skip-contents: true`(as we saw in the [docs](https://www.aptly.info/doc/aptly/publish/repo/)), in the `PUT` call, as part of the final package step.But this also seemed to not work.

After digging a bit more, [Vidit](https://twitter.com/viditganpi/) and [Kartik](https://twitter.com/kartik7153/) discovered that we were passing the wrong header key which would be used to Skip contents, from their test suite, the configuration to allow [setting this value](https://github.com/aptly-dev/aptly/blob/24a027194ea8818307083396edb76565f41acc92/system/t12_api/publish.py#L49), the header to be passed was `SkipContents: true`, after making the change. This was the step which was taking all the time as the package index size grew over time, making the whole package upload step slow.

## Alternative ways of tackling the package index step slowdown

One route, which can be taken would be, to create a new aptly instance, start pushing your debian package files to this instance instead rather than the older slower one.

How would the client pick up packages from both these apt sources? Multiple sources can be specified in files under `/etc/apt/sources.list.d/`, which will be looked up while searching for a package. For newer packages, the client will pick it up from the newer apt source (new aptly instance) and for the older ones, it will pick it from the older apt source (older aptly instance)

Will immediately solve the problem of slower builds, as well as the sideeffect of having a clean newer setup, where you can start enforcing the standard of keeping foo number of packages from now on

The other solution is to either self host another alternative like Pulp3 etc, or a paid package manager, which would take away some bits of these off your plate.

Another thing to note here is that, aptly hasn't had a commit on it's master for quite some time along with a new release not being put our for some time now. There has been an open [issue](https://github.com/aptly-dev/aptly/issues/920) regarding the question of whether it's maintained anymore. Although, I personally feel it's feature complete for the set of features we have been currently using, and runs without any fuss whatsover for the most part, this is definitely something which you should consider as something while weighing down on options.

## References

- [https://www.aptly.info/doc/aptly/publish/repo/](https://www.aptly.info/doc/aptly/publish/repo/)
- [https://github.com/aptly-dev/aptly/blob/24a027194ea8818307083396edb76565f41acc92/system/t12_api/publish.py#L49](https://github.com/aptly-dev/aptly/blob/24a027194ea8818307083396edb76565f41acc92/system/t12_api/publish.py#L49)
- [https://github.com/aptly-dev/aptly/issues/920](https://github.com/aptly-dev/aptly/issues/920)

## Credits

- Post header image credits: www.aptly.info
