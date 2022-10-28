---
layout: post
title: "Golang setup for vim"
description: "Golang setup for vim"
tags: [golang, vim]
comments: true
share: true
cover_image: ''
---

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Cleaning my vim config after some time and ended up removing a bunch of things and starting afresh, a couple of plugins which had been archived but worked all the while were <a href="https://t.co/lmjXtN7HbS">https://t.co/lmjXtN7HbS</a>, switched to <a href="https://t.co/7ZSoKMgwBW">https://t.co/7ZSoKMgwBW</a> as it was the recommended replacement (1/n)</p>&mdash; Tasdik Rahman (@tasdikrahman) <a href="https://twitter.com/tasdikrahman/status/1585959045122621440?ref_src=twsrc%5Etfw">October 28, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Cleaned up my [vim](https://www.vim.org/) configuration which I was trying to set up for golang specifically. I did have [vim-go](https://github.com/fatih/vim-go) already setup, but wanted to try out [coc.vim](https://github.com/neoclide/coc.nvim) along with [gopls](https://github.com/golang/tools/tree/master/gopls) since it was around for sometime now. There were also a bunch of plugins which had been archived like [syntastic](https://github.com/vim-syntastic/syntastic)

This setup will evolve and has space for a couple of things which I will incrementally add over time in the next few weeks.

# ale along vim.coc

I turned off the lsp settings for ale, so that vim.coc would take it over

# Impressions so far

The jump of definitions after moving over to gopls, and vim.coc giving over jumping to references, definitions, godocs comes almost the same to the setup for an IDE, along with being able to fold along with using [nvim-treesitter](https://github.com/nvim-treesitter/nvim-treesitter) is something which adds up really well along with [ctrl+p](https://github.com/kien/ctrlp.vim)

# Changes and things kept from existing vim-go setup

Turned off the go autoimports functionality which is given over by vim-coc

The go-build and go-run keybindings which I had are already being used, same for the :GoAlternate keybindings to jump and split the panes when going over test and implementation files, along with the highlight configs is something which I have kept.

I did end up removing ultisnips which is one of the plugins presented in vim-go, with coc-go and coc-snippets, as they almost provide the same thing.

# tl;dr what does all this get me in my setup

- jump to definitions
- jump to references
- jump to symbols
- fuzzy file search
- code folding
- jumping between test and implementation file
- testing specific function
- real time code linting

# What does it lack as of now

I want to setup up delve as that's what I feel is remaining as of now for a full out IDE setup which is what I was anyways using, this remains as of now. Will see how [nvim-dap](https://github.com/mfussenegger/nvim-dap) fixes up with my current setup and update this section with what I ended up with.

# Links

The latest vim config being used by me and what I did to set it up can be found here in the following link.

- [https://github.com/tasdikrahman/dotfiles/tree/master/vim](https://github.com/tasdikrahman/dotfiles/tree/master/vim)
