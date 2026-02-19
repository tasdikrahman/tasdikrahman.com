---
title: "AddTrust Root expiration fix"
description: "If you are affected by the Sectigo root cert"
tags: [ssl]
comments: true
share: true
cover_image: '/content/images/2020/05/internet.jpg'
---

With the root cert expiring for [sectigo](https://support.sectigo.com/articles/Knowledge/Sectigo-AddTrust-External-CA-Root-Expiring-May-30-2020), the older linux distributions are not properly ignoring the cert.

I have seen this affect boxes which ran ubuntu 16.04, but there would be others too. Didn't notice anything on Debian 10(buster)

As people have pointed out [around](https://www.reddit.com/r/linux/comments/gshh70/sectigo_root_ca_expiring_may_not_be_handled_well/), 
this is an openssl 1.0.2 bug. So even a system upgrade wouldn't help the situation wouldn't help, as this would require
an actual distro upgrade.

Programs which don't depend on openssl(like go binaries), won't get affected by this. Services/client on Ruby/Jruby for example, on the other hand will have problems similar to curl.

The same goes for programs which do certificate pinning on their clients. Personally, saw a saas vendor dole out a fix for this yesterday for their python client, These clients would see external calls to other endpoints failing.

This is also a great twitter thread by [Ryan](https://twitter.com/sleevi_/status/1266647545675210753) on twitter

## How curl fails in a typical affected client box

```
$ curl https://myremoteserver.com
curl: (60) server certificate verification failed. CAfile: /etc/ssl/certs/ca-certificates.crt CRLfile: none
More details here: http://curl.haxx.se/docs/sslcerts.html

curl performs SSL certificate verification by default, using a "bundle"
 of Certificate Authority (CA) public keys (CA certs). If the default
 bundle file isn't adequate, you can specify an alternate file
 using the --cacert option.
If this HTTPS server uses a certificate signed by a CA represented in
 the bundle, the certificate verification probably failed due to a
 problem with the certificate (it might be expired, or the name might
 not match the domain name in the URL).
If you'd like to turn off curl's verification of the certificate, use
 the -k (or --insecure) option.
```

## What should I do if I am affected by this?

- the option to upgrade your distro is more or less out of the picture if you are currently affected by this, as you would need a tactical fix. This 
being more of a long term strategic fix. 
- [Andrew](https://www.agwa.name/blog/post/fixing_the_addtrust_root_expiration) mentioned a fix which would involve putting a ! before mozilla/AddTrust Root cert
- [kingsly](https://twitter.com/kingslyj) suggested this fix where we do a 


```
echo -n > /usr/share/ca-certificates/mozilla/AddTrust_Low-Value_Services_Root.crt;\
echo -n > /usr/share/ca-certificates/mozilla/AddTrust_Public_Services_Root.crt; \
echo -n > /usr/share/ca-certificates/mozilla/AddTrust_Qualified_Certificates_Root.crt;\ 
echo -n > /usr/share/ca-certificates/mozilla/AddTrust_External_Root.crt;\
update-ca-certificates
```

The above would empty out the contents of the file, you could further use [chattr +i on the file](https://linux.die.net/man/1/chattr) to change the file attributes such 
that the above files are not modified from the state which we have set them to, but the bad part of the changing this attribute is that when do a system update,
this file would not get updated. 

- A less disruptive approach, for debian based systems as pointed out by [shani](https://github.com/shanipribadi) would be to do a 

```
sudo sed -i -e 's/^mozilla\/AddTrust_External_Root.crt/!mozilla\/AddTrust_External_Root.crt/' /etc/ca-certificates.conf
sudo update-ca-certificates --fresh
```
