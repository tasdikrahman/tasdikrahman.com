---
title: "Migrating this blog from Jekyll to Hugo"
description: "Notes on moving tasdikrahman.com from Jekyll to Hugo"
tags: [blogmaintenance]
comments: true
share: true
cover_image: ''
---

## Why I started looking

So it all started with me wanting to update the dependencies of my [Jekyll](https://jekyllrb.com/) website. I hadn't really updated any dependencies for my website since ages and after I had changed my machine, I hadn't also played around with ruby/ruby on rails, so it just started with installing Ruby. Yes, you got it right, I was just practically pushing markdown files for quite some time for even years maybe(?) at this point for the blog. It's also very beautiful that GitHub just allows building such an old Jekyll website. This website is dated from 2015 but I've obviously updated my bundle dependencies and gems, maybe a couple of times since then. While trying to install this I also got stuck with the dependency hell loop. Which made me pause further.

I had also recently installed Claude Code on my local machine, so also wanted to take it for a spin. I had already been using GitHub Copilot but I feel I was missing out on the whole agentic flow of doing things. That made me look further into the direction of researching a bit further on what other options I had, and Hugo came on top of the list. I think I saw [Zola](https://www.getzola.org/) also as one of the alternatives and a couple more. Given Hugo's huge ecosystem and a couple of my friends also using the same, I decided that it was probably the simplest approach to migrate.

## Migration effort

To be honest I'm very impressed. I started about an hour and a half or somewhere around that to migrate my Jekyll blog and ended up migrating the whole thing and then actually publishing the website in less than, I think, close to two hours from when I started. I'm pretty sure it would have taken me longer than that to probably migrate if I was sitting at one stretch without the assistance of AI here, which brings me to the point that I feel, for a migration like this, Claude Code/the agentic AI approach really assisted and accelerated my workflow.

## What was done

Note: Used AI to summarise this section of the blog

- Migrated from Jekyll 3.8.5 (via the `github-pages` gem) to Hugo v0.156.0 with the PaperMod theme, dropping the entire Ruby/Bundler dependency chain
- Scaffolded Hugo in-place using `hugo new site . --force` alongside existing Jekyll files, allowing incremental migration without losing history
- Added PaperMod as a git submodule, pinned to a version requiring Hugo >= 0.146.0
- Moved 95 posts from `_posts/` to `content/posts/` — front matter was largely compatible, only stripped `layout: post` and enabled filename-based date inference (`[frontmatter] date = [":filename", ":default"]`) as a fallback for posts without explicit `date:` fields
- Enabled raw HTML rendering in Goldmark (`[markup.goldmark.renderer] unsafe = true`) to preserve YouTube iframes and SpeakerDeck embeds embedded in posts
- Enabled JSON output alongside HTML and RSS for Fuse.js-powered client-side search
- Moved `content/images/` to `static/content/images/` so all existing image paths in posts remained unchanged
- Migrated static pages (About, Speaking, Projects, Resume) and added new Hugo-native pages: Archive (posts grouped by year/month), Search, and Tags
- Replaced the `github-pages` gem's Jekyll-based deployment with a GitHub Actions workflow using the modern GitHub Pages approach — `actions/configure-pages`, `actions/upload-pages-artifact`, and `actions/deploy-pages` — triggered on push to `main` with no separate `gh-pages` branch needed
- Fixed several broken links carried over from Jekyll: tag URLs (`/blog/tag/` → `/tags/`), malformed anchor `name` attributes, a double `http://http://` typo, and stale relative links pointing to old Jekyll pages

## Farewell Jekyll, hello Hugo

<center><img src="/content/images/2026/02/old-jekyll-website.png" alt="The old Jekyll blog"></center>

I will probably write another post on my current and past AI workflows. As of now I just want to thank deeply the creators of Jekyll for making it so seamless to have allowed me to blog since 2015, from my university days until now. [This is how the blog used to look](https://web.archive.org/web/20250305193726/https://tasdikrahman.com/) before the migration. It marks the end of around close to 11 years of blogging with jekyll. I must say that it was very seamless, apart from the bundle dependency stuff which you would have to deal with. In my honest opinion I think this was probably one of the best decisions that I had made when I started blogging, which allowed me to write as seamlessly as possible and without too many interruptions. I'm looking forward to using Hugo and playing around with it; and hopefully it also gives this same seamless experience.

For reference, here's the
- First PR which does the migration https://github.com/tasdikrahman/tasdikrahman.com/pull/1
- and the PR which removes all the old jekyll file formats https://github.com/tasdikrahman/tasdikrahman.com/pull/1
- and the first hugo blog deployment of my website https://github.com/tasdikrahman/tasdikrahman.com/actions/runs/22202652165
