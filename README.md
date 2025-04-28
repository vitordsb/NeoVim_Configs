-- init.lua - Configuração organizada para Neovim com lazy.nvim
-- Arquivo: ~/.config/nvim/init.lua

-- ==============================
-- Seção 1: Configurações Gerais
-- ==============================

-- 1.1 Definir leader keys
vim.g.mapleader = " "
vim.g.maplocalleader = " "

-- 1.2 Opções básicas do editor
local o = vim.opt
o.number = true                     -- mostrar número absoluto
o.relativenumber = true             -- mostrar número relativo
o.clipboard = "unnamedplus"       -- usar clipboard do sistema
o.expandtab = true                  -- converter tab em espaços
o.shiftwidth = 2                    -- indent de 2 espaços
o.tabstop = 2                       -- tab = 2 espaços
o.smartindent = true                -- indent inteligente
o.wrap = false                      -- sem wrap de linha
o.mouse = "a"                      -- habilitar mouse
o.termguicolors = true              -- habilitar truecolor
o.completeopt = "menuone,noselect"-- comportamento do autocomplete
o.laststatus = 0                    -- esconde a statusline inferior

-- ==============================
-- Seção 2: Bootstrap do lazy.nvim
-- ==============================

local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system({
    "git", "clone", "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git",
    "--branch=stable", lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)

-- ==============================
-- Seção 3: Declaração de Plugins
-- ==============================

require("lazy").setup({
  -- Tema VSCode (semelhante ao React Theme)
  {
    "Mofiqul/vscode.nvim",
    config = function()
      require("vscode").setup({})
      vim.cmd("colorscheme vscode")
    end,
  },

  -- Utilitários Lua
  { "nvim-lua/plenary.nvim" },

  -- Mason para gerenciar LSPs/CLIs
  { "williamboman/mason.nvim" },
  { "williamboman/mason-lspconfig.nvim" },

  -- Configuração de LSP
  { "neovim/nvim-lspconfig" },

  -- Autocomplete e Snippets (nvim-cmp + LuaSnip)
  {
    "hrsh7th/nvim-cmp",
    dependencies = {
      "hrsh7th/cmp-nvim-lsp",
      "hrsh7th/cmp-buffer",
      "hrsh7th/cmp-path",
      "hrsh7th/cmp-cmdline",
      "saadparwaiz1/cmp_luasnip",
      "L3MON4D3/LuaSnip",
      "rafamadriz/friendly-snippets",
    },
    config = function()
      local cmp = require("cmp")
      local luasnip = require("luasnip")
      require("luasnip.loaders.from_vscode").lazy_load()
      cmp.setup({
        snippet = { expand = function(args) luasnip.lsp_expand(args.body) end },
        mapping = cmp.mapping.preset.insert({
          ["<C-Space>"] = cmp.mapping.complete(),
          ["<Tab>"] = cmp.mapping(function(fallback)
            if cmp.visible() then cmp.select_next_item()
            elseif luasnip.expand_or_jumpable() then luasnip.expand_or_jump()
            else cmp.complete() end
          end, {"i","s"}),
          ["<S-Tab>"] = cmp.mapping(function(fallback)
            if cmp.visible() then cmp.select_prev_item()
            elseif luasnip.jumpable(-1) then luasnip.jump(-1)
            else fallback() end
          end, {"i","s"}),
          ["<CR>"] = cmp.mapping.confirm({ select = true }),
        }),
        sources = cmp.config.sources({
          { name = "nvim_lsp" },
          { name = "buffer" },
          { name = "path" },
        }),
        completion = { autocomplete = { cmp.TriggerEvent.TextChanged } },
      })
      -- enable cmp in cmdline
      cmp.setup.cmdline("/", { mapping = cmp.mapping.preset.cmdline(), sources = {{ name = "buffer" }} })
      cmp.setup.cmdline(":", { mapping = cmp.mapping.preset.cmdline(), sources = cmp.config.sources({{ name = "path" }}, {{ name = "cmdline" }}) })
    end,
  },

  -- Treesitter (sintaxe avançada)
  { "nvim-treesitter/nvim-treesitter", build = ":TSUpdate" },
  { "nvim-treesitter/playground" },
  { "windwp/nvim-ts-autotag", dependencies = { "nvim-treesitter/nvim-treesitter" }, config = function()
      require("nvim-ts-autotag").setup()
    end
  },

  -- Autoclose brackets/pairs
  {
    "windwp/nvim-autopairs",
    config = function()
      require("nvim-autopairs").setup({
        check_ts = true,
        disable_filetype = {"TelescopePrompt", "vim"},
      })
    end,
  },

  -- Emmet (HTML/CSS/React snippets)
  {
    "mattn/emmet-vim",
    ft = { "html", "css", "javascriptreact", "typescriptreact", "vue" },
    config = function()
      vim.g.user_emmet_install_global = 0
      vim.g.user_emmet_leader_key = '<Tab>'
    end,
  },
  {
    "dcampos/cmp-emmet-vim",
    dependencies = { "hrsh7th/nvim-cmp", "mattn/emmet-vim" },
    config = function()
      require("cmp").setup.buffer({
        sources = require("cmp").config.sources({{ name = "emmet_vim" }}, {{ name = "nvim_lsp" }, { name = "buffer" }, { name = "path" }})
      })
    end,
  },

  -- Windsurf (antigo Codeium) - AI Assistant
  {
    "Exafunction/windsurf.nvim",
    dependencies = { "nvim-lua/plenary.nvim", "hrsh7th/nvim-cmp" },
    config = function()
      require("codeium").setup({
        -- opcional: configurar opções como virtual_text, map_keys etc.
      })
    end,
  },

  -- Telescope (busca fuzzy)
  { "nvim-telescope/telescope.nvim", dependencies = { "nvim-lua/plenary.nvim" } },

  -- Git integration
  { "tpope/vim-fugitive" },

  -- Árvore de arquivos à direita
  {
    "nvim-tree/nvim-tree.lua",
    dependencies = { "nvim-tree/nvim-web-devicons" },
    config = function()
      require("nvim-tree").setup({ view = { width = 30, side = "right" } })
    end,
  },

  -- Terminal integrado (ToggleTerm)
  {
    "akinsho/toggleterm.nvim",
    version = "*",
    config = function()
      require("toggleterm").setup({
        size = 20,
        open_mapping = [[<C-j>]],
        direction = "horizontal",
        close_on_exit = true,
        insert_mappings = true,
      })
    end,
  },
})

-- ==============================
-- Seção 4: Mapeamentos de Teclas
-- ==============================

local map = vim.api.nvim_set_keymap
local opts = { noremap = true, silent = true }
map("n", "<C-b>", "<cmd>NvimTreeToggle<cr>", opts)
map("n", "<C-p>", "<cmd>Telescope find_files<cr>", opts)

vim.opt.clipboard = "unnamedplus"

-- ==============================
-- Seção 5: Auto Commands
-- ==============================

vim.api.nvim_create_autocmd("BufWritePost", {
  pattern = vim.fn.stdpath("config") .. "/init.lua",
  command = "source <afile> | Lazy sync",
})

-- ==============================
-- Seção 6: Configurações de Plugins Extras
-- ==============================

require("nvim-treesitter.configs").setup({ ensure_installed = { "c", "lua", "python", "javascript", "html", "css" }, highlight = { enable = true } })

-- ==============================
-- Seção 7: Configuração de LSP e Mason
-- ==============================

require("mason").setup()
require("mason-lspconfig").setup({ ensure_installed = { "pyright", "ts_ls", "html", "cssls", "jsonls" } })
local capabilities = require("cmp_nvim_lsp").default_capabilities()
local lspconfig = require("lspconfig")
for _, srv in ipairs({ "pyright", "ts_ls", "html", "cssls", "jsonls" }) do
  lspconfig[srv].setup({ capabilities = capabilities })
end

