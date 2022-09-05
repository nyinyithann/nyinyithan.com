---
layout: post
title: NeoVim setup in Lua for OCaml development
categories: [NeoVim, OCaml]
tags: [NeoVim, OCaml]
---

In this post, I'll describe how I configured NeoVim in Lua for OCaml development.
First of all, NeoVim and OCaml need to be installed. Please check out the following links if you haven't already installed them.
- NeoVim: [https://github.com/neovim/neovim/wiki/Installing-Neovim](https://github.com/neovim/neovim/wiki/Installing-Neovim)
- OCaml: [https://ocaml.org/docs/up-and-running](https://ocaml.org/docs/up-and-running)

The following OCaml packages are necessary to be installed.
- [ocaml-lsp-server](https://opam.ocaml.org/packages/ocaml-lsp-server/) 
- [ocamlformat](https://opam.ocaml.org/packages/ocamlformat/)
- [ocamlformat-rpc](https://opam.ocaml.org/packages/ocamlformat-rpc/)
```bash
opam install ocaml-lsp-server
opam install ocamlformat
opam install ocamlformat-rpc
```

### Essential Lua
Lua is used in the entire setup, and in this section, I'll highligh the essential knowledge of Lua required for the setup.

Lua has the following types:
- `nil` — represents the absence of a useful value
- `boolean` — true and false
- `number` — double-precision floating-point numbers
- `string` — immutable sequence of bytes
- `table` — associative arrays
- `userdata` — pointer to a block of raw memory
- `thread` —  represents independent threads of execution

`userdata` and `thread` are not used in this setup.

Lua has global and local variables. `nil` is the default value of any non-initialized variables.
```lua
local x = 10 -- local variable
y = 10 -- here y is global variable
local z -- z is nil
```

Lua has `table` type which implements associative array. Lua `table` can be used as an array or a record. 
If it's used as an array, index starts at 1.
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

Lua has a function called `require` which can be used to load libraries or modules.
Let's say I have a file called "utilities.lua", and I can import it into "init.lua" by calling `require("utilities")` inside it.

### Folder structure


