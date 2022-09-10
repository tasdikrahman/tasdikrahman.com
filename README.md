## Blog

Based on [Pool/Hyde](https://github.com/poole/hyde)

Heavily influnced from 

- [chris banes blog](https://github.com/chrisbanes/chrisbanes.github.io)
- [fred's blog](https://fredrikaverpil.github.io)

Hosted with love on Github pages, Domain registered at [Namecheap](https://namecheap.com)

## setup

#### OS X

```
$ git clone https://github.com/tasdikrahman/www.tasdikrahman.com && cd www.tasdikrahman.com
$ git checkout gh-pages
$ brew install ruby coreutils
$ gem install jekyll bundler
$ bundle install
$ bundle exec jekyll build 
$ bundle exec jekyll serve
```

Go to [http://localhost:4000/](http://localhost:4000/) and voila!

**NOTE**: Don't use `sudo` if you don't wan't to break things for other users on the machine.
