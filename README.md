## Blog

Built with [Hugo](https://gohugo.io) and the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme.

Hosted with love on Github Pages, domain registered at [Namecheap](https://namecheap.com)

## Setup

### Prerequisites

Install Hugo extended (v0.146.0 or later):

```bash
# Download the latest binary and place in ~/bin
wget https://github.com/gohugoio/hugo/releases/latest/download/hugo_extended_linux-amd64.tar.gz -O /tmp/hugo.tar.gz
tar -xzf /tmp/hugo.tar.gz -C /tmp hugo
mv /tmp/hugo ~/bin/hugo
```

### Run locally

```bash
git clone https://github.com/tasdikrahman/tasdikrahman.com && cd tasdikrahman.com
git submodule update --init --recursive
hugo server -D
```

Go to [http://localhost:1313/](http://localhost:1313/) and voila!

### Build

```bash
hugo --minify
```

Output is generated in the `public/` directory.

## Deployment

The site is automatically built and deployed to GitHub Pages via GitHub Actions on every push to the `source` branch.
