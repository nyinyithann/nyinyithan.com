---
layout: post
title: NeoVim setup in Lua for OCaml development
categories: [NeoVim, OCaml]
tags: [NeoVim, OCaml]
---

In this post, I'll describe how I configured NeoVim in Lua for OCaml development.
First of all, NeoVim and OCaml need to be installed. Please check out the following links if you haven't already installed them.
- <a href="https://github.com/neovim/neovim/wiki/Installing-Neovim" target="_blank"> 📝 NeoVim Install<a/>
- <a href="https://ocaml.org/docs/up-and-running" target="_blank">  🐫 OCaml Install </a>
- <a href="https://www.nerdfonts.com" target="_blank">  🔠 Nerd Fonts</a><br/>You may use installation script [here](https://github.com/ronniedroid/getnf) to install Nerd Fonts.

After installation above, the following OCaml packages will be installed.
- [ocaml-lsp-server](https://opam.ocaml.org/packages/ocaml-lsp-server/) 
- [ocamlformat](https://opam.ocaml.org/packages/ocamlformat/)
- [ocamlformat-rpc](https://opam.ocaml.org/packages/ocamlformat-rpc/)
```bash
opam install ocaml-lsp-server
opam install ocamlformat
opam install ocamlformat-rpc
```

### Essential Lua
Lua is used in the entire setup, and in this section I'll highligh the essential knowledge of Lua required for the setup.

Lua has the following types:
- `nil` — represents the absence of a useful value
- `boolean` — true and false
- `number` — double-precision floating-point numbers
- `string` — immutable sequence of bytes
- `table` — associative arrays
- `userdata` — pointer to a block of raw memory
- `thread` —  represents independent threads of executions

`'userdata'` and `'thread'` are not used in this setup.

Lua has global and local variables. `'nil'` is the default value of any non-initialized variables.
```lua
local x = 10 -- local variable
y = 10 -- here y is global variable
local z -- z is nil
```

Lua has `'table'` type which implements associative array. It can be used as an array or a record. 
Customarily Lua array index starts at 1. But you can start an array at index 0, 1, -1 , or any other value.
```lua
local t = {}  -- create an empty table

-- table as an array 
local vowels = { "a", "e", "i", "o", "u" }
print(vowels[0]) --> nil
print(vowels[1]) --> "a"  (array index starts at 1)

-- table as a record (key/value pairs)
local person = { 
  ["name"] = "Pip", 
  ["age"] = 10 
}
print(person["name"]) --> "Pip"
print(person["age"]) --> 10

-- table as a class
local Point = { x = 4.2, y = 4.5 }

function Point:getX () 
    return self.x
end

function Point:setX(v) 
    self.x = v
end

print(Point:getX()) --> 4.2
Point:setX(7.4)
print(Point:getX()) --> 7.4


-- or like this
local tbl = { "x", "y", "z", ["name"] = "Neo", ["age"] = 20 }
print(tbl[1]) --> x
print(tbl[4]) --> nil
print(tbl["name"]) --> "Neo"
```

Lua has a function called `'require'` which can be used to load libraries or modules.
Let's say I have a file called `'utilities.lua'`, and I can import it into `'init.lua'` by calling `'require("utilities")'` inside it.<br/>

In Lua, `..` is string concatenation operator. And multi-line string is written in double square brackets.
```lua
print ("Hello," .. " World!") --> "Hello, World!"
local multiline = [[to vim
or not to vim
that's the question
]] 
print (multiline) 
```
Multi-line string is mostly used to call Vimscript from Lua like below:
```lua
vim.cmd [[
 " equalalize split views
 augroup _auto_resize
    autocmd!
    autocmd VimResized * tabdo wincmd =
  augroup end
]]
```
`'vim.cmd'` is a function, and in Lua you can call a function without parenthesis if it has only one argument which is either a literal string or a table. So `'vim.cmd "echo hello"'` is same as `'vim.cmd ("echo hello")'` or `'vim.cmd [[echo hello]]'`.

### Folder structure
By default, Nvim's user-specific configuration file is located at `'~/.config/nvim/init.lua'` and it will be loaded everytime Nvim starts. The folder structure of my configurations is as below. Basic configuration files are in `'nvim/lua'` folder. All the plugin configurations are under `'nvim/after/plugin'` and they are loaded by Nvim automatically. 
```
nvim
├── after
│   └── plugin
│       ├── bufferline.lua
│       ├── catppuccin.lua
│       ├── cmp.lua
│       ├── indentation.lua
│       ├── lspconfig.lua
│       ├── lualine.lua
│       ├── lunasnip.lua
│       ├── symbols.lua
│       ├── telescope.lua
│       ├── toggleterm.lua
│       ├── tree.lua
│       └── treesitter.lua
├── init.lua
├── lua
    ├── autocommands.lua
    ├── colorscheme.lua
    ├── disable.lua
    ├── maps.lua
    ├── options.lua
    ├── plugins.lua
    └── utilities.lua
```
### Basic configurations
`'init.lua'` loads all the modules defined in `'nvim/lua'` folder.
```lua
-- init.lua
require("options")
require("disable")
require("maps")
require("autocommands")
require("colorscheme")
require("plugins")
require("utilities")
```
`'options.lua'` contains Nvim option setttings. You can check out my settings at <a href="https://github.com/nyinyithann/dotfiles/blob/main/mac/nvim_lua/lua/options.lua" target="_blank">options.lua.</a><br/>
`'disable.lua'` disables some built-in features I don't need. More at <a href="https://github.com/nyinyithann/dotfiles/blob/main/mac/nvim_lua/lua/disable.lua" target="_blank">disable.lua.</a><br/>
`'maps.lua'` is for custom key bindings. More at <a href="https://github.com/nyinyithann/dotfiles/blob/main/mac/nvim_lua/lua/maps.lua" target="_blank">maps.lua.</a><br/>
I define some autocommands in <a href="https://github.com/nyinyithann/dotfiles/blob/main/mac/nvim_lua/lua/autocommands.lua" target="_blank">`'autocommands.lua'`</a>. An autocommand executes vim actions in response to an event.<br/>

### Plugin manager
I use <a href="https://github.com/wbthomason/packer.nvim" target="_blank">packer</a> for plugin management. Plugin management settings is configured in `'lua/plugins.lua'` as below. You may copy the following into your `'plugins.lua'` file, save it, close and reopen Nvim, and Packer will be installed. Then Packer will sync all the plugins mentioned in the function passed to `'packer.startup'`. Everytime you add a new plugin into it and when you save the file, Packer will automatically install the plugin. You will see a popup dialog when Packer is installing plugins. You need to press 'q' to close the popup when the installation is done. 
```lua
-- plugins.lua 

local fn = vim.fn
-- Automatically install packer
local install_path = fn.stdpath("data") .. "/site/pack/packer/start/packer.nvim"
if fn.empty(fn.glob(install_path)) > 0 then
    PACKER_BOOTSTRAP = fn.system({
        "git",
        "clone",
        "--depth",
        "1",
        "https://github.com/wbthomason/packer.nvim",
        install_path,
    })
    print("Installing packer close and reopen Neovim...")
    vim.cmd([[packadd packer.nvim]])
end

-- Autocommand that reloads neovim whenever you save the plugins.lua file
vim.cmd([[
  augroup packer_user_config
    autocmd!
    autocmd BufWritePost plugins.lua source <afile> | PackerSync
  augroup end
]])

-- Use a protected call so we don"t error out on first use
local status_ok, packer = pcall(require, "packer")
if not status_ok then
    print("Packer is not installed")
    return
end

-- Have packer use a popup window
packer.init({
    display = {
        open_fn = function()
            return require("packer.util").float({ border = "rounded" })
        end,
    },
})

packer.startup(function(use)
    -- packer can manage itself!
    use "wbthomason/packer.nvim"

    -- nvim-tree file explorer
    use {
        "kyazdani42/nvim-tree.lua",
        requires = { "kyazdani42/nvim-web-devicons" }
    }

    -- lsp config
    use { "neovim/nvim-lspconfig", tag = "v0.1.3" }

    -- colorscheme
    use { "catppuccin/nvim", as = "catppuccin" }

    -- autocompletion
    use "hrsh7th/nvim-cmp"
    use "hrsh7th/cmp-nvim-lsp"
    use "hrsh7th/cmp-buffer"
    use "hrsh7th/cmp-nvim-lsp-signature-help"
    use "hrsh7th/cmp-path"
    use "hrsh7th/cmp-nvim-lua"
    use "saadparwaiz1/cmp_luasnip"
    use "rafamadriz/friendly-snippets"

    -- snippet
    use "L3MON4D3/LuaSnip"

    -- symbol
    use "simrat39/symbols-outline.nvim"

    -- statusbar
    use {
        "nvim-lualine/lualine.nvim",
        requires = { "kyazdani42/nvim-web-devicons", opt = true }
    }

    -- bufferline
    use { "akinsho/nvim-bufferline.lua", tag = "v2.7.0" }

    -- treesitter
    use {
        "nvim-treesitter/nvim-treesitter",
        run = function() require("nvim-treesitter.install").update({ with_sync = true }) end,
    }

    -- terminal
    use { "akinsho/toggleterm.nvim", tag = "v2.*" }

    -- telescope
    use {
        "nvim-telescope/telescope.nvim", tag = "0.1.0",
        requires = { { "nvim-lua/plenary.nvim" } }
    }
    use { "nvim-telescope/telescope-fzf-native.nvim", run = "make" }

    -- indentation guides
    use "lukas-reineke/indent-blankline.nvim"

    -- Automatically run packer.clean() followed by packer.update() after cloning packer.nvim
    -- Put this at the end after all plugins
    if PACKER_BOOTSTRAP then
        require("packer").sync()
    end
end)
```

### File explorer, Status bar, and Tab
I am using `'nvim-tree'` for file explorer. The configuration is in `'tree.lua'` file.
```lua
-- tree.lua
local status, tree = pcall(require, "nvim-tree")
if (not status) then return end
local lib = require "nvim-tree.lib"
-- [credit] https://github.com/alex-courtis/arch/blob/c91322f51c41c2ace04614eff3a37f13e5f22a24/config/nvim/lua/nvt.lua
local function cd_dot_cb(node)
    tree.change_dir(vim.fn.getcwd(-1))
    if node.name ~= ".." then
        lib.set_index_and_redraw(node.absolute_path)
    end
end
tree.setup({
    sort_by = "case_sensitive",
    open_on_setup = true,
    view = {
        adaptive_size = false,
        mappings = {
            list = {
                { key = "u", action = "dir_up" },
                { key = ".", action = "cd_dot", action_cb = cd_dot_cb, },
                { key = "v", action = "vsplit" },
                { key = "h", action = "split" },
            },
        }
    },
    renderer = {
        icons = {
            glyphs = {
                git = {
                    unstaged = "⊛"
                }
            }
        }
    },
    filters = {
        -- hide .git folder
        custom = { "^\\.git$" }
    },
    diagnostics = {
        enable = true,
        show_on_dirs = true,
        icons = {
            error = " ",
            warning = " ",
            info = " ",
            hint = " "
        },
    },
})
local opts = { noremap = true, silent = true }
vim.keymap.set("n", "<space>te", "<Cmd>NvimTreeToggle<CR>", opts)
vim.keymap.set("n", "<space>tf", "<Cmd>NvimTreeFocus<CR>", opts)
```
