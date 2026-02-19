---
title: "The making of bhola - your cert expiration overseer - Part 1"
description: "The making of bhola - your cert expiration overseer - Part 1"
tags: [ruby, rubyonrails]
comments: true
share: true
cover_image: '/content/images/2020/10/bhola.png'
---

You might have already seen me writing a bit about bhola already on [twitter](https://twitter.com/tasdikrahman), I wrote a little bit about why I have been building [bhola](https://github.com/tasdikrahman/bhola). This post is more of a continuation to this tweet and what I envision it to be moving forward.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Do you sometimes wake up, with a call by someone from your team, telling you some SSL cert has expired? Do you keep track of SSL cert expirations on your to do notes or excel sheets? Would you like to be on top of such x509 cert renewals? <a href="https://t.co/MVFRZCUlZN">https://t.co/MVFRZCUlZN</a> is for you (1/n) <a href="https://t.co/pj8JHJEkje">pic.twitter.com/pj8JHJEkje</a></p>&mdash; Tasdik Rahman (@tasdikrahman) <a href="https://twitter.com/tasdikrahman/status/1306945863369936896?ref_src=twsrc%5Etfw">September 18, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## What was the inspiration

Do you sometimes wake up, with a call by someone from your team, telling you some SSL cert has expired? Do you keep track of SSL cert expirations on your to do notes or excel sheets? Would you like to be on top of such x509 cert renewals?

All of this are directly due to
- No visibility about when the certificate is expiring
- No alerts in form of email or test message when the certificate is expiring

Then [bhola](https://github.com/tasdikrahman/bhola) is for you!

## What does bhola do?

[v0.1](https://github.com/tasdikrahman/bhola/releases/tag/v0.1.0) of Bhola, gives you a dead simple API, which you can use to ask Bhola, to track domains which have certs attached to it. It automatically checks for the cert expiration in the background keeping note of when is it expiring.

The operator can set a buffer period, which would bhola, then use to see if it meets the threshold number of days, before the cert is going to expire, before marking the cert, that it needs renewal asap.

Want to check, what domains, is it already tracking? Bhola comes with a very simple bare minimum UI, which one can use to check, what domains are being tracked, when are they expiring and other metadata details, like who issuer of the domain, when is it expiring, when was it issued.

What is required by Bhola to run? It just needs a good old postgres to function to keep track of the domains, and that's it. Nothing shiny. Plain and simple.

Further on [v0.2](https://github.com/tasdikrahman/bhola/releases/tag/v0.2.0) of bhola adds support for sending notifications to slack as part of evolving, from just being a dashboard to something which can be used to preemptively alert the operator on is the certificate expiring.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">It will alert for all the domains, which have already expired/are about to expire within the buffer period which you have set &amp; send notification to your slack channel via webhook endpoint, periodically checking in the interval set by the operator, for expiration. (2/n) <a href="https://t.co/IdpDGxJQtr">pic.twitter.com/IdpDGxJQtr</a></p>&mdash; Tasdik Rahman (@tasdikrahman) <a href="https://twitter.com/tasdikrahman/status/1312414866133512192?ref_src=twsrc%5Etfw">October 3, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Further more, smaller improvements like 1 step dev setup, docker-compose setup and container images available for docker would make reproducing the setup for bhola easier than before, than the status in milestone 0.1.

Not that it matters much, but [rails](https://rubyonrails.org/) has been a fun framework to work on, especially with [rspec](https://rspec.info/), practicing [BDD](https://en.wikipedia.org/wiki/Behavior-driven_development) and [TDD](https://en.wikipedia.org/wiki/Test-driven_development) has been a great experience.

### How does one even insert a domain to be tracked as of now?

##### Example request
```
$ curl --location --request POST 'localhost:3000/api/v1/domains' \
  --header 'Content-Type: application/json' \
  --data-raw '{
      "fqdn": "https://expired.badssl.com"
  }'
```

##### Example response
```
{
    "data": {
        "fqdn": "expired.badssl.com",
        "certificate_expiring": true,
        "certificate_issued_at": "2016-08-08T21:17:05.000Z",
        "certificate_expiring_at": "2018-08-08T21:17:05.000Z",
        "certificate_issuer": "/C=US/ST=California/L=San Francisco/O=BadSSL/CN=BadSSL Intermediate Certificate Authority"
    },
    "errors": []
}
```
#### querying the domains stored
##### Example request
```
$ curl --location --request GET 'localhost:3000/api/v1/domains' \
  --header 'Accept: application/json'
```

##### Example response
```
{
    "data": [
        {
            "fqdn": "www.tasdikrahman.com",
            "certificate_expiring": false,
            "certificate_issued_at": "2020-05-06T00:00:00.000Z",
            "certificate_expiring_at": "2022-04-14T12:00:00.000Z",
            "certificate_issuer": "/C=US/O=DigiCert Inc/OU=www.digicert.com/CN=DigiCert SHA2 High Assurance Server CA"
        },
        {
            "fqdn": "expired.badssl.com",
            "certificate_expiring": true,
            "certificate_issued_at": "2016-08-08T21:17:05.000Z",
            "certificate_expiring_at": "2018-08-08T21:17:05.000Z",
            "certificate_issuer": "/C=US/ST=California/L=San Francisco/O=BadSSL/CN=BadSSL Intermediate Certificate Authority"
        }
    ],
    "errors": []
}
```

### Are there other ways to do such domain expiration checks?

Yes, there are are other ways to do this.

If you are on [cert-manager](https://github.com/jetstack/cert-manager/), then you can make use of [https://grafana.com/grafana/dashboards/11001](https://grafana.com/grafana/dashboards/11001), adding alerts on top of the dashboard should not be very complicated.

There is an exporter [https://github.com/ribbybibby/ssl_exporter](https://github.com/ribbybibby/ssl_exporter), which will do the scraping for you and expose the expiration date of the cert as the metric `ssl_cert_not_after`, which then you can add an alert on top.

```
scrape_configs:
  - job_name: "ssl"
    metrics_path: /probe
    static_configs:
      - targets:
          - example.com:443
          - prometheus.io:443
```

Where `example.com` and `prometheus.io` would be your scrape endpoints, in this example.

Thanks to [Joy](https://twitter.com/hashfyre/), for pointing the above out to me.

Then there is a [guide](https://www.robustperception.io/get-alerted-before-your-ssl-certificates-expire), on how to do with prometheus, via the the [blackbox exporter](https://github.com/prometheus/blackbox_exporter)

And the good old `openssl` command will always be there

```bash
$ echo | openssl s_client -servername www.tasdikrahman.com -connect www.tasdikrahman.com:443 2>/dev/null | openssl x509 -noout -dates
notBefore=Aug 19 13:59:14 2020 GMT
notAfter=Nov 17 13:59:14 2020 GMT
```

### How does me running bhola help then?

The above tools will work, no doubt about it, if you already are on these systems, you can definitely leverage them to make use of them in ways similarly shown above. But even then, redundancy is never a bad thing to have.

Adding to it, one plus which bhola has is the validations before tracking endpoints, not tracking invalid/not having certs attached to endpoints. Which helps in keeping the entries sane.

If you are using [letsencrypt](https://letsencrypt.org/), given that the certificates would be expiring within [3 months](https://community.letsencrypt.org/t/pros-and-cons-of-90-day-certificate-lifetimes/4621) and that they also send [email notifications](https://letsencrypt.org/docs/expiration-emails/), you would have used some automation to renew your certificates, due to the 3months expiration policy of LE certs. Having bhola as your external system to monitor your domains, would be an extra guard against the automation failing silently or the emails getting missed.

Furthermore, bhola panders more to the userbase, who are just in search of something, running which they can just start tracking and getting alerts for their domains, rather than tinkering with tools which they may be unfamiliar with, hence further reducing their friction in prioritizing their efforts to add alerting on domain expirations, rather than first trying to run [prometheus](https://github.com/prometheus/prometheus)(if they aren't already) or for those who don't yet have the right level of automation maturity for their certificate renewals. If you/your org are already on a level where you have already done and dusted this alerting and cert renewal part via one of the ways described above or via some other way, then if I may say, you would come under a minority and not the norm.

Albeit there is an overhead with bhola which will be running a rails service and a postgres db. The decision then would with the operators, on what kind of solution would they be comfortable maintaining. People should always have multiple options for such tools to pick with, depending on their comfort level and the level of overhead they would want in their system.

### Assumptions made by bhola

- bhola assumes that the dns being inserted, resolves to a single IP, so in case you are doing dns loadbalancing on a single FQDN, with multiple IP's behind it, it may try connecting to whichever IP first get's returned.
- bhola will not register the domain to be tracked, if it can't reach it, it would be apt to place bhola somewhere, in your network, which would make it possible for bhola to resolve your dns endpoints with ease, so in case, the domains which you are trying to track, if they resolve to a private IP, make sure bhola can reach them.
- bhola will not register the domain to be tracked, if it doesn't have an SSL cert attached, it will not track it.

### What bhola will not be

- will not generate certificates for you by being the intermediate broker
- will not install the certificates for it's clients
- will not provide a UI to generate/install/replace the certs for it's clients

### What's next?

I envision bhola to be a 1 stop service for your needs of tracking your domain expirations for starters and there are quite a few thing which I want to see in it, in future. Some of them being

- Ability for it to associate domains and alerts with users, this will allow bhola to be multi-tenant.
  - The idea is to have a system in place if someone wants to enable this feature, this can be turned on with just enabling a feature flag when they start the
    the webserver.
  - while accessing the api to do insertion for tracking domains, add authz/authn.
- Ability to delete domains associated with a user.
- Ability to sign up using email id.
- Ability to send alert notifications of all the domains associated to the user in their specified email id.
- Not an immediate goal, but I want to host this on my own infrastructure, as a public facing endpoint.

While I have not spread the above in specific milestones, but I would mostly pick up the first one for milestone 0.3.

As bhola is completely open source, would love to hear what you feel can be added to make [bhola](https://github.com/tasdikrahman/bhola) better than before.

