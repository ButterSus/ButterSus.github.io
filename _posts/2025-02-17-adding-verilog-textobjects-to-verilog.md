---
layout: post
title: "SystemVerilog setup for NeoVim: treesitter-textobjects"
tags: neovim
---

NeoVim provides a powerful plugin [nvim-treesitter-textobjects](https://github.com/nvim-treesitter/nvim-treesitter-textobjects). By default, it provides its own textobjects:
- Classes
- Functions
- Blocks
- Comments
- Conditionals
- etc.

The only issue is that most of these _are not implemented_ in most languages, including SystemVerilog. This post shows how to add support for SystemVerilog textobjects.

# Setup

> If you're only interested in code or an example,
please check out [my gist](https://gist.github.com/ButterSus/5e1fff58b15e8efc854ff16983e228e4). Feel free to ask any questions.

## Install nvim-treesitter-textobjects

If you're using [LazyVim](http://www.lazyvim.org/), just add the following plugin entry:

```lua
-- treesitter.lua
return {
  -- TreeSitter plugin
  {
    "nvim-treesitter/nvim-treesitter",
    event = { "BufReadPost", "BufNewFile" },
    opts = {
      ensure_installed = {
        "verilog"  -- Add Verilog/SystemVerilog support
      },
      textobjects = {
        ...  -- Your own config
      }
    },
    config = function(_, opts)
      require("nvim-treesitter.configs").setup(opts)
    end
  },

  -- To Support textobjects
  {
    "nvim-treesitter/nvim-treesitter-textobjects",
    dependencies = "nvim-treesitter/nvim-treesitter",
  }
}
```

Now you can use builtin textobjects in SystemVerilog files.
However, you need to add keymaps & additional textobjects first.

## Configure treesitter-textobjects

Check out an [official](https://github.com/nvim-treesitter/nvim-treesitter-textobjects?tab=readme-ov-file#text-objects-select) example and [configuration](https://raw.githubusercontent.com/nvim-treesitter/nvim-treesitter-textobjects/refs/heads/master/doc/nvim-treesitter-textobjects.txt).

Or take this as an example:

```lua
-- Inside of your treesitter.lua
textobjects = {
  select = {
    enable = true,
    lookahead = true,  -- You need this to achieve correct behaviour
    include_surrounding_whitespace = function(a)
      -- I highly recommend defining a function to check if surrounding whitespace is needed
      if a.query_string == "@parameter.outer" then
        return true
      elseif a.query_string == "@loop.outer" then
        return true
      elseif a.query_string == "@conditional.outer" then
        return true
      elseif a.query_string == "@block.outer" then
        return true
      elseif a.query_string == "@assignment.outer" then
        return true
      elseif a.query_string == "@module.outer" then
        return true
      elseif a.query_string == "@macro.outer" then
        return true
      elseif a.query_string == "@call.outer" then
        return true
      elseif a.query_string == "@comment.outer" then
        return true
      elseif a.query_string == "@statement.outer" then
        return true
      elseif a.query_string == "@function.outer" then
        return true
      elseif a.query_string == "@class.outer" then
        return true
      else
        return false
      end
    end,
  }
  keymaps = {  -- These are my keymaps, customize to your needs
    ["af"] = "@function.outer",
    ["if"] = "@function.inner",
    ["ac"] = "@class.outer",
    ["ic"] = "@class.inner",
    ["aa"] = "@parameter.outer",
    ["ia"] = "@parameter.inner",
    ["al"] = "@loop.outer",
    ["il"] = "@loop.inner",
    ["ai"] = "@conditional.outer",
    ["ii"] = "@conditional.inner",
    ["av"] = "@field.outer",
    ["iv"] = "@field.inner",
    ["ab"] = "@block.outer",
    ["ib"] = "@block.inner",
    ["is"] = "@statement.inner",
    ["as"] = "@statement.outer",
    ["aq"] = "@call.outer",
    ["iq"] = "@call.inner",
    ["i/"] = "@comment.inner",
    ["a/"] = "@comment.outer",
    ["aA"] = "@assignment.outer",
    ["iA"] = "@assignment.inner",
    ["iH"] = "@assignment.lhs",
    ["iL"] = "@assignment.rhs",
    ["im"] = "@module.inner",
    ["am"] = "@module.outer",
    ["id"] = "@macro.inner",
    ["ad"] = "@macro.outer",
  }
}
```

Optionally, you can add move motions (look at [gist](https://gist.github.com/ButterSus/5e1fff58b15e8efc854ff16983e228e4) for full code):

```lua
-- Inside of your treesitter.lua
move = {
  enable = true,
  set_jumps = true,
  goto_next_start = {
    ["]f"] = "@function.outer",
    ["]c"] = "@class.outer",
    ["]a"] = "@parameter.outer",
    ["]l"] = "@loop.outer",
    ["]i"] = "@conditional.outer",
    ["]v"] = "@field.outer",
    ["]b"] = "@block.outer",
        ... -- and so on
```

## Overriding Verilog Textobjects

Create file `<nvim-config>/queries/verilog/textobjects.scm` with this [code](https://gist.githubusercontent.com/ButterSus/5e1fff58b15e8efc854ff16983e228e4/raw/c0acc4c4562aef9b447f18af1d3b08e26f666754/textobjects.scm).

That's it! Now you can use treesitter-textobjects in Verilog files.

- However, please note that each node defined affects overall performance, so make sure
  to delete all the S-expressions that you don't need. For example:
  ```scm
  DELETE STARTING HERE
  ;; Macros  HERE'S A BLOCK OF S-EXPRESSIONS
  ;; ======

  ; DEFINE
  ; ------

  ; Outer
  (text_macro_definition) @macro.outer  HERE IS AN S-EXPRESSION

  ; Inner
  (text_macro_definition
    . (text_macro_name)
    . (macro_text) @macro.inner)
  DELETE ENDING HERE
  ```

