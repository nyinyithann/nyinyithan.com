---
layout: post
title: NeoVim setup in Lua for OCaml development
categories: [NeoVim, OCaml]
tags: [NeoVim, OCaml]
---

In this post, I'll describe how I configured NeoVim in Lua for OCaml development. The configuration will give me Nvim looking like below.

<img src="{{ site.baseurl }}/assets/nvimsetup/main.jpg" alt="drawing" height="500"/>

First of all, the followings need to be installed.

- <a href="https://github.com/neovim/neovim/wiki/Installing-Neovim" target="_blank"> 📝 NeoVim Installation</a> ( NeoVim >= 0.7  should be installed.)
- <a href="https://www.nerdfonts.com" target="_blank">  🔠 Nerd Fonts</a>  Installation script [here](https://github.com/ronniedroid/getnf).
- <a href="https://ocaml.org/docs/up-and-running" target="_blank">  🐫 OCaml Installation </a>
- The following OCaml packages
  - [ocaml-lsp-server](https://opam.ocaml.org/packages/ocaml-lsp-server/) 
  - [ocamlformat](https://opam.ocaml.org/packages/ocamlformat/)
  - [ocamlformat-rpc](https://opam.ocaml.org/packages/ocamlformat-rpc/)
    ```bash
    opam install ocaml-lsp-server
    opam install ocamlformat
    opam install ocamlformat-rpc
    ```

### Essential Lua
Lua is used for the configuration, and in this section I'll highligh the essential Lua required for the setup.

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

-- both numeric and string index
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
By default, Nvim's user-specific configuration file is located at `'~/.config/nvim/init.lua'` and it will be loaded everytime Nvim starts. The folder structure of my configurations is as below. Basic configuration files are in `'nvim/lua'` folder. All the plugin configurations are in `'nvim/after/plugin'` and they are loaded by Nvim automatically. 
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
│       ├── symbols.lua
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
```
### Basic configurations
`'init.lua'` loads all the modules defined in `'nvim/lua'` folder.
```lua
-- init.lua
require("disable")
require("options")
require("maps")
require("autocommands")
require("colorscheme")
require("plugins")
```
`'options.lua'` contains Nvim option setttings. You can check out my settings at <a href="https://github.com/nyinyithann/dotfiles/blob/main/mac/nvim_lua/lua/options.lua" target="_blank">options.lua.</a><br/>
`'disable.lua'` disables some built-in features I don't need. More at <a href="https://github.com/nyinyithann/dotfiles/blob/main/mac/nvim_lua/lua/disable.lua" target="_blank">disable.lua.</a><br/>
`'maps.lua'` is for custom key bindings. More at <a href="https://github.com/nyinyithann/dotfiles/blob/main/mac/nvim_lua/lua/maps.lua" target="_blank">maps.lua.</a><br/>
I define some autocommands in <a href="https://github.com/nyinyithann/dotfiles/blob/main/mac/nvim_lua/lua/autocommands.lua" target="_blank">autocommands.lua</a>. An autocommand executes vim actions in response to an event.<br/>
I set the theme in <a href="" target="_blank">colorscheme.lua</a>. I am using `'catppuccin'` theme.

### Plugin manager
I use <a href="https://github.com/wbthomason/packer.nvim" target="_blank">packer</a> for plugin management. Plugin management settings is configured in `'lua/plugins.lua'` as below. You may copy the following into your `'plugins.lua'` file, save it, close and reopen Nvim, and Packer will be installed. Then Packer will sync all the plugins mentioned in the function passed to `'packer.startup'`. You will see a popup dialog when Packer is installing plugins. You need to press 'q' to close the popup when the installation is done. Everytime you add a new plugin or remove an existing one from `'plugins.lua'`, Packer will automatically install or remove the plugin as soon as you save the file. 
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
    -- statusbar
    use {
        "nvim-lualine/lualine.nvim",
        requires = { "kyazdani42/nvim-web-devicons", opt = true }
    }
    -- bufferline
    use { "akinsho/nvim-bufferline.lua", tag = "v2.7.0" }
    -- colorscheme
    use { "catppuccin/nvim", as = "catppuccin" }
    -- lsp config
    use { "neovim/nvim-lspconfig", tag = "v0.1.3" }
 		-- treesitter
    use {
        "nvim-treesitter/nvim-treesitter",
        run = function() require("nvim-treesitter.install").update({ with_sync = true }) end,
    }
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
		-- indentation guides
    use "lukas-reineke/indent-blankline.nvim"
    -- terminal
    use { "akinsho/toggleterm.nvim", tag = "v2.*" }
    -- Automatically run packer.clean() followed by packer.update() after cloning packer.nvim
    -- Put this at the end after all plugins
    if PACKER_BOOTSTRAP then
        require("packer").sync()
    end
end)
```

### Nvim-tree, Lualine, Bufferline
I am using `'nvim-tree'` for file explorer, `'lualine'` for status bar, and `'bufferline'` for showing opening buffer names and buffer/tab management. My configurations for them can be found at [nvim-tree](), [lualine](), [bufferline]().  You may also have a look at my [catppuccin theme configuration]() for integration with `nvim-tree` and `bufferline`. Based on my configuration, i will get Nvim looking like this.

<img src="{{ site.baseurl }}/assets/nvimsetup/explore-status-tab.jpg" alt="drawing" width="600" height="500"/>

### Configuring Language Server Client

Nvim provides a language server protocol (LSP) client.  I use `'lspconfig'` plugin to configure the client to consume services from OCaml language server. Basically the setup is something like below. You may check out the complete configuration of [my lspconfig here]().

```lua
-- lspconfig.lua

local status, lsp = pcall(require, "lspconfig")
if (not status) then return end

local on_attach = function(client, bufnr)
    -- enable completion triggered by <C-x><C-o>
    vim.api.nvim_buf_set_option(bufnr, "omnifunc", "v:lua.vim.lsp.omnifunc")

    local bufopts = { noremap=true, silent=true, buffer=bufnr }
    vim.keymap.set("n", "gD", vim.lsp.buf.declaration, bufopts)
    vim.keymap.set("n", "gd", vim.lsp.buf.definition, bufopts)
    vim.keymap.set("n", "K", vim.lsp.buf.hover, bufopts)
    vim.keymap.set("n", "gi", vim.lsp.buf.implementation, bufopts)
    vim.keymap.set("n", "gk", vim.lsp.buf.signature_help, bufopts)
    vim.keymap.set("n", "<space>wa", vim.lsp.buf.add_workspace_folder, bufopts)
    vim.keymap.set("n", "<space>wr", vim.lsp.buf.remove_workspace_folder, bufopts)
    vim.keymap.set("n", "<space>wl", function()
         print(vim.inspect(vim.lsp.buf.list_workspace_folders()))
    end, bufopts)
    vim.keymap.set("n", "<space>td", vim.lsp.buf.type_definition, bufopts)
    vim.keymap.set("n", "<space>rn", vim.lsp.buf.rename, bufopts)
    vim.keymap.set("n", "<space>ca", vim.lsp.buf.code_action, bufopts)
    vim.keymap.set("n", "gr", vim.lsp.buf.references, bufopts)
    vim.keymap.set("n", "<space>f", vim.lsp.buf.formatting, bufopts)
    vim.keymap.set("n", "<space>do", vim.diagnostic.open_float, bufopts)
    vim.keymap.set("n", "gp", vim.diagnostic.goto_prev, bufopts)
    vim.keymap.set("n", "gl", vim.diagnostic.goto_next, bufopts)
    vim.keymap.set("n", "<space>dl", vim.diagnostic.setloclist, bufopts)

    -- format on save
    if client.server_capabilities.documentFormattingProvider then
        vim.api.nvim_create_autocmd("BufWritePre", {
            group = vim.api.nvim_create_augroup("Format", { clear = true }),
            buffer = bufnr,
            callback = function() vim.lsp.buf.formatting_seq_sync() end
        })
    end

    -- code lens 
    if client.resolved_capabilities.code_lens then
        local codelens = vim.api.nvim_create_augroup(
            'LSPCodeLens',
            { clear = true }
        )
        vim.api.nvim_create_autocmd({ 'BufEnter', 'InsertLeave', 'CursorHold' }, {
            group = codelens,
            callback = function()
                vim.lsp.codelens.refresh()
            end,
            buffer = bufnr,
        })
    end
end

local c = vim.lsp.protocol.make_client_capabilities()
c.textDocument.completion.completionItem.snippetSupport = true
c.textDocument.completion.completionItem.resolveSupport = {
    properties = {
        'documentation',
        'detail',
        'additionalTextEdits',
    },
}
local capabilities = require("cmp_nvim_lsp").update_capabilities(c)

lsp.ocamllsp.setup({
    cmd = { "ocamllsp" },
    filetypes = { "ocaml", "ocaml.menhir", "ocaml.interface", "ocaml.ocamllex", "reason", "dune" },
    root_dir = lsp.util.root_pattern("*.opam", "esy.json", "package.json", ".git", "dune-project", "dune-workspace"),
    on_attach = on_attach,
    capabilities = capabilities
})
```

With the configuration, I would get Nvim looking like below when i open OCaml project.

<img src="{{ site.baseurl }}/assets/nvimsetup/ocamlsp.jpg" alt="drawing" width="600" height="500"/>

### Autocomplete

I am using `'nvim-cmp'` for autocompletion. `'vim-cmp'` needs a snippet source otherwise it might throw error sometimes. I use `'luasnip'` for snippet engine and you might notice it's already added into `'plugins.lua'` file. The following is the configuration of `'vim-cmp'`. You may also have a look at [cmp.lua]().

```lua
-- cmp.lua

local status, cmp = pcall(require, "cmp")
if (not status) then return end

local luasnip = require("luasnip")
local lsp = require("lspconfig")

local has_words_before = function()
    local line, col = unpack(vim.api.nvim_win_get_cursor(0))
    return col ~= 0 and vim.api.nvim_buf_get_lines(0, line - 1, line, true)[1]:sub(col, col):match("%s") == nil
end

local cmp_kinds = {
    Text = "",
    Method = "",
    Function = "",
    Constructor = "",
    Field = "",
    Variable = "",
    Class = "ﴯ",
    Interface = "",
    Module = "",
    Property = "ﰠ",
    Unit = "",
    Value = "λ",
    Enum = "",
    Keyword = "",
    Snippet = "",
    Color = "",
    File = "",
    Reference = "",
    Folder = "",
    EnumMember = "",
    Constant = "",
    Struct = "",
    Event = "",
    Operator = "",
    TypeParameter = ""
}

cmp.setup({
    snippet = {
        expand = function(args)
            luasnip.lsp_expand(args.body)
        end,
    },
    window = {
        completion = {
            winhighlight = "Normal:Pmenu,FloatBorder:Pmenu,Search:None",
        },
        documentation = {
            border = { "╭", "─", "╮", "│", "╯", "─", "╰", "│" },
        },
    },
    mapping = cmp.mapping.preset.insert({
        ["<C-k>"] = cmp.mapping.scroll_docs(-4),
        ["<C-j>"] = cmp.mapping.scroll_docs(4),
        ["<C-e>"] = cmp.mapping.abort(),
        ["<CR>"] = cmp.mapping.confirm({
            behavior = cmp.ConfirmBehavior.Replace,
            select = true
        }),
        ["<Tab>"] = cmp.mapping(function(fallback)
            if cmp.visible() then
                cmp.select_next_item()
            elseif luasnip.expand_or_jumpable() then
                luasnip.expand_or_jump()
            elseif has_words_before() then
                cmp.complete()
            else
                fallback()
            end
        end, { "i", "s" }),
        ["<S-Tab>"] = cmp.mapping(function(fallback)
            if cmp.visible() then
                cmp.select_prev_item()
            elseif luasnip.jumpable(-1) then
                luasnip.jump(-1)
            else
                fallback()
            end
        end, { "i", "s" }),
    }),
    sources = cmp.config.sources({
        { name = "nvim_lsp" },
        { name = "luasnip" },
        { name = "buffer" },
        option = {
            get_bufnrs = function()
                local bufs = {}
                for _, win in ipairs(vim.api.nvim_list_wins()) do
                    bufs[vim.api.nvim_win_get_buf(win)] = true
                end
                return vim.tbl_keys(bufs)
            end
        }
    }),
    formatting = {
        fields = { "kind", "abbr", "menu" },
        format = function(entry, vim_item)
            vim_item.kind = cmp_kinds[vim_item.kind] or ""
            local lsp_icon = "🅻"
            if lsp ~= nil and lsp.ocamllsp ~= nil then
                lsp_icon = "🐫"
            end
            vim_item.menu = ({
                buffer = "🅱",
                nvim_lsp = lsp_icon,
                luasnip = "㊊"
            })[entry.source.name]
            return vim_item
        end,
    }
})
```

With the above configuration, autocompletion will look like below:

<img src="{{ site.baseurl }}/assets/nvimsetup/autocomplete.jpg" alt="drawing" width="600" height="400"/>

### Treesitter, Indentation, and Symbols

I am using `'treesitter'`for better syntax highlighting. You may use the following configuration for it. After saving the config, you need to close and reopen Nvim to install OCaml parser.

```lua
-- config file name: tree.lua

local status, ts = pcall(require, "nvim-treesitter.configs")
if (not status) then return end

ts.setup {
    highlight = {
        enable = true,
        disable = {},
    },
    indent = {
        enable = true,
        disable = {},
    },
    ensure_installed = {
        "ocaml",
    },
    sync_install = true,
    autotag = {
        enable = true,
    },
    rainbow = {
        enable = true,
        extended_mode = true,
        max_file_lines = nil,
    },
    autopairs = {
        enable = true,
    }
}
```

For indentation line and symbol outline, i use `'indent-blankline'` and `'symbols-outline'` plugin. Please check out [indentation.lua]() and [symbols.lua]() for the configuration. The configruations give me Nvim looking like this:

<img src="{{ site.baseurl }}/assets/nvimsetup/treesitter.jpg" alt="drawing" width="600" height="400"/>

The GitHub repo related to this post is [here](https://github.com/nyinyithann/neovim-ocaml).

Or If you want to check out my Nvim setup for OCaml, Rust, Rescript, JavaScript, TypeScript, and TailwindCSS, plese have a look at [here](https://github.com/nyinyithann/dotfiles/tree/main/mac/nvim_lua).

Thank you for reading the post.
