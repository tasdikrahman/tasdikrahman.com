---
title: "neovim setup for golang 6 months in"
description: "neovim setup for golang 6 months in"
tags: [golang,vim]
comments: true
share: true
cover_image: ''
---

# So far

I have been using my nvim golang setup for around more than 6months now and I am still learning something new everyday while I use it.

coc.vim works really well so far for me. Linting, autocomplete, jumping to definitions back and forth, code folding, checking for references for where a function/method is used, it works out for me well so far.

# What I am trying to fix on my setup so far

The debugging experience for sure can be improved, I have tried using [nvim-dap-go](https://github.com/leoluz/nvim-dap-go/).

It does work for tests where you would not have a lot of nesting, for example works great for simple `t.Run()` calls which are not overly nested (at least from my usage so far) but I have had trouble using it with some [ginkgo](https://github.com/onsi/ginkgo) style tests where it's not able to parse for the parent test when trying to add a debug point, internally it ends up using treesitter to parse the whole file.

Have an open issue here which I added more context in this issue [https://github.com/leoluz/nvim-dap-go/issues/9#issuecomment-1521496733](https://github.com/leoluz/nvim-dap-go/issues/9#issuecomment-1521496733).

My setup for the same is very simple and mostly untouched from the usual config which is out there in the docs


```vim
lua <<EOF
local dap = require('dap')
local dapui = require("dapui")

dapui.setup()

require('dap-go').setup {
  -- delve configurations
  delve = {
      -- time to wait for delve to initialize the debug session.
      -- default to 20 seconds
      initialize_timeout_sec = 20,
      -- a string that defines the port to start delve debugger.
      -- default to string "${port}" which instructs nvim-dap
      -- to start the process in a random available port
      port = "${port}"
  },
  dap_configurations = {
    {
      type = "go",
      name = "Attach remote",
      mode = "remote",
      request = "attach",
    },
  },
}

-- You can use nvim-dap events to open and close the windows automatically
-- from: https://github.com/rcarriga/nvim-dap-ui
dap.listeners.after.event_initialized["dapui_config"] = function()
  dapui.open()
end
dap.listeners.before.event_terminated["dapui_config"] = function()
  dapui.close()
end
dap.listeners.before.event_exited["dapui_config"] = function()
  dapui.close()
end

EOF
```

# Workflow

The flow of usage is pretty out of the box
- add your breakpoints in the place you want with `:lua require'dap'.toggle_breakpoint()` at the places you want
- then debug a specific test with `:lua require('dap-go').debug_test()`
- to go over through the breakpoints, you do a `:lua require('dap').continue()`
- and to terminate the process `lua require'dap'.terminate()`

# Key maps being used

I am as of now using the following keymaps

```vim
nmap <silent> <leader>tb :lua require'dap'.toggle_breakpoint()<CR>
nmap <silent> <leader>tc :lua require'dap'.continue()<CR>
nmap <silent> <leader>tt :lua require'dap'.terminate()<CR>
nmap <silent> <leader>td :lua require('dap-go').debug_test()<CR>
```
