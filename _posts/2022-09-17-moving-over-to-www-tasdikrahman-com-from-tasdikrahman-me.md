---
layout: post
title: "Moving over to www.tasdikrahman.com from tasdikrahman.me"
description: "Moving over to www.tasdikrahman.com from tasdikrahman.me"
tags: [blog]
comments: true
share: true
cover_image: ''
---

I have been writing over at this blog since mid 2015 and not much has changed over these years on this blog, the same theme, the same static website generater, the same color scheme.

## Why did I end up moving?

The .me domain name, from what I could gather, the registry which is operating it has access to it only till 2023, not that I am anticipating anything out of the blue, but I felt it's just makes sense long term to move to .com.

Plus, the .me domain was with me since it came free along with the github education pack back in the days in college, which is why my domain tasdikrahman.me existed in the first place, I never ended up moving away from it at that time.

## What did I end up doing

### Add redirect on the old page

Looking around, since the page was statically generated, I found it just easier to add a block

```html
<meta http-equiv="Refresh" content="0; url='https://www.tasdikrahman.com'" />
```

in the `head.html` for the older website which I was having, since each generated html page would need to have it, ended up adding it in `head.html` [here](https://github.com/tasdikrahman/tasdikrahman.me/commit/f67b2110789427bc9f43b073c66f796512e83737)

There are more ways to do the same, one of them being https://about.txtdirect.org/hosted/, which would make use to TXT records, and act as the middle layer, to redirect back to wherever you want it to go.

I didn't want to make use of an external service/entity at this point unless really required, hence didn't end up going in that direction, although if you have other suggestions, would love to hear about them.

### Adding redirect for tasdikrahman.com to redirect to www.tasdikrahman.com

This was simple and easy, adding a temporaty redirect on the domain registrar (google domains in this case) was simple and straight forward.

Enabling SSL on for the same again super simple, as github pages gives the option to enable it straight on the settings of the repo from where the static website is being generated from.

## Ending notes

I have always loved the simplicity of static website generators, the fact that I haven't changed anything for so long, and the add markdown, commit changes, git push model reducing the barrier to writing for me is a huge plus for me.

Github pages exposing the github actions used to build and deploy the pages also makes it easier in case I want to move to another static website generator at some point. Plus this made me realize that the blog has grown over to [~100MB](https://github.com/tasdikrahman/tasdikrahman.com/actions/runs/3028630771) as of now.

