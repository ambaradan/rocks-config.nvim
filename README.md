<!-- vale off-->
<!-- markdownlint-disable -->
<br />
<div align="center">
  <a href="https://github.com/nvim-neorocks/rocks-config.nvim">
    <img src="./rocks-header.svg" alt="rocks-config.nvim">
  </a>
  <p align="center">
    <!-- <br /> -->
    <!-- <a href="./doc/rocks-config.txt"><strong>Explore the docs »</strong></a> -->
    <!-- <br /> -->
    <br />
    <a href="https://github.com/nvim-neorocks/rocks-config.nvim/issues/new?assignees=&labels=bug">Report Bug</a>
    ·
    <a href="https://github.com/nvim-neorocks/rocks-config.nvim/issues/new?assignees=&labels=enhancement">Request Feature</a>
    ·
    <a href="https://github.com/nvim-neorocks/rocks.nvim/discussions/new?category=q-a">Ask Question</a>
  </p>
  <p>
    <strong>
      Permettete a <a href="https://github.com/nvim-neorocks/rocks.nvim/">rocks.nvim</a> di aiutarvi a configurare i vostri plugin!
    </strong>
  </p>
</div>
<!-- markdownlint-restore -->

[![LuaRocks][luarocks-shield]][luarocks-url]

## :star2: Sommario

`rocks-config.nvim` estende [`rocks.nvim`](https://github.com/nvim-neorocks/rocks-config.nvim)
con la possibilità di configurare i vostri plugin.

## :hammer: Installazione

È sufficiente eseguire `:Rocks install rocks-config.nvim`,
e siete pronti a partire!

## :books: Uso

Con questa estensione, è possibile aggiungere una tabella `[config]` al proprio `rocks.toml`,
for example:

```toml
[plugins]
"neorg" = "7.0.0"
"sweetie.nvim" = "1.0.0"

[config]
plugins_dir = "plugins/"
auto_setup = false
```

### Opzioni

#### `plugins_dir`

La sottodirectory (relativa a `nvim/lua`, predefinita: `plugins`)
per cercare le configurazioni dei plugin. È possibile aggiungere una cartella `lua/plugins/`
alla configurazione di `nvim`, con uno script lua per ogni plugin.

```sh
── nvim
  ├── lua
  │  └── plugins # Your plugin configs go here.
  │     └── neorg.lua
  │     └── sweetie.lua # or sweetie-nvim.lua
  ├── init.lua
```

All'avvio, per ogni plugin presente nel file `rocks.toml` `[plugins]`
questo modulo cercherà un modulo di configurazione corrispondente in
e caricarlo se ne viene trovato uno.

> [!NOTE]
>
> I nomi dei file di configurazione possibili sono:
>
> - Il nome del plugin (purché sia un nome di modulo valido).
> - Il nome del plugin con il suffisso `.[n]vim` rimosso[^1].
> - Il nome del plugin con il prefisso `[n]vim-` rimosso[^2].
> - Il nome del plugin con `"-"` sostituito da `"."`[^3].

[^1]: Per esempio, un file di configurazione per un plugin chiamato `foo.nvim` potrebbe essere chiamato `foo.lua`.
[^2]: Per esempio, un file di configurazione per un plugin chiamato `nvim-foo` potrebbe essere chiamato `foo.lua`.
[^3]: Per esempio, un file di configurazione per un plugin chiamato `foo.bar` potrebbe essere chiamato `foo-bar.lua`.

Se si disinstalla un plugin, è possibile lasciare la sua configurazione (ad esempio nel caso in cui
si desidera reinstallarlo in un secondo momento), e non causerà alcun
problema.

#### `<plugin>.config` - Adding basic configurations to rocks.toml

Molti plugin di Neovim richiedono una chiamata a una funzione `setup`,
che in genere prende una tabella di configurazione.
Se nessuna delle opzioni di configurazione è una funzione lua,
è possibile aggiungere la configurazione a rocks.toml e questo plugin
chiamerà automaticamente `setup` con le opzioni del plugin.

Ad esempio, la seguente configurazione lua:

```lua
-- lua/plugins/lualine.lua
require('lualine').setup {
  options = {
    icons_enabled = true,
    theme = 'auto',
  },
}
```

... può essere configurato anche tramite rocks.toml:

```toml
# rocks.toml
[plugins.lualine.config]
options = { icons_enabled = true, theme = "auto" }
```

Il campo `config' può anche assumere la forma di una stringa che punta a un modulo Lua (se si vuole
come eseguire uno script una tantum in una posizione diversa dall'opzione globale `plugins_dir`):

```toml
# rocks.toml
[plugins.lualine]
config = "plugins.statusline" # Eseguirà `.config/nvim/lua/plugins/statusline.lua
```

#### `auto_setup`

Alcuni plugin che non funzionano senza una chiamata a `setup`,
anche se si è soddisfatti delle opzioni predefinite.
`rocks-config.nvim` fornisce un hack per aggirare questo problema
con l'opzione `auto_setup` (disabilitata per impostazione predefinita).
Se abilitato, non viene trovata alcuna configurazione per un plugin installato,
questo modulo tenterà di chiamare `require('<nome-plugin>').setup()`
for you.

> [!WARNING]
>
> L'abilitazione di `auto_setup` potrebbe portare a comportamenti inaspettati.
> Per esempio, se un plugin che non ha bisogno di una chiamata `setup`
> ha una logica di configurazione/inizializzazione nel suo modulo principale,
> sarà invocato con la chiamata a `require`,
> potenzialmente si ottiene un'inizializzazione più avida del necessario.

È anche possibile attivare/disattivare `auto_setup` per singoli plugin
impostando rispettivamente `config = true` o `config = false`.

### Ordine di inizializzazione

rocks.nvim utilizza la [sequenza di inizializzazione] integrata di Neovim (https://neovim.io/doc/user/starting.html#initialization),
e fornisce un gancio che rocks-config.nvim utilizza per caricare le configurazioni
*prima* che gli script dei plugin siano originati[^4].

[^4]: Le API lua di tutti i plugin sono disponibili non appena lo snippet
dal programma di installazione rocks.nvim è stato eseguito.

Se si ha bisogno di reperire gli script di un plugin in modo avventato (ad esempio,
per caricare i temi colore), è possibile impostare il plugin `opt = true` in
rocks.toml, e poi caricarlo con `vim.cmd.Rocks({"packadd", "<nome-rock>"})` [^5]


[^5]: Vedi `:h rocks.commands`
[^6]: Vedi `:h rocks.lua`

## Pacchetti di plugin

Oltre alla configurazione per singolo plugin, è anche possibile creare collezioni di plugin
(called bundles) and configure them all in one go.

Di seguito è riportato un esempio che raggruppa i plugin relativi all'LSP:

```toml
[plugins]
"neodev.nvim"= "scm"
nvim-lspconfig = { version = "0.1.7" } 

[plugins.nvim-cmp]
git = "hrsh7th/nvim-cmp" # Use the git version of nvim-cmp for the best experience.

[bundles.lsp] # Create a bundle called `lsp`
items = [
    "neodev.nvim",
    "nvim-lspconfig",
    "nvim-cmp"
]
```

Ora, invece di richiamare ogni singolo file di configurazione per ogni plugin separatamente,
`rocks-config.nvim` cercherà invece un file `lua/plugins/lsp.lua` che verrà eseguito.


#### `config`

Analogamente ai normali plugin, un campo `config` può essere applicato a un bundle. Questo campo `config`
può essere solo una stringa che punta a un modulo Lua alternativo da eseguire. Esempio:

```toml
[bundles.lsp] # Create a bundle called `lsp`
items = [
    "neodev.nvim",
    "nvim-lspconfig",
    "nvim-cmp"
]
config = "bundles.language_support"
```

Invece di cercare all'interno di `lua/plugins/lsp.lua`, `rocks-config.nvim` ora cercherà
un file in `lua/bundles/language_support.lua`.

#### `load_opt_plugins`

Per impostazione predefinita, `rocks-config.nvim` non carica le configurazioni di

- plugin con `opt = true
- o i plugin che sono stati contrassegnati come `opt', ad esempio da rocks-lazy.nvim.

Inoltre, non caricherà i bundle che contengono un plugin che corrisponde ai criteri di cui sopra.

È possibile sovrascrivere questo comportamento impostando `load_opt_plugins = true`,
oppure si può caricare la configurazione di un plugin usando la funzione `configure(name)`:

```lua
require("rocks-config").configure("foo.nvim")
vim.cmd.packadd("foo.nvim")
```

## Configurazione Neovim

Si può anche usare `rocks-config.nvim` per impostare varie opzioni di Neovim
nel vostro rocks.toml.

Ecco un esempio:

```toml
[config]
colorscheme = "kanagawa"

[config.options]
number = true
relativenumber = true
hlsearch = true
completeopt = 'menu,menuone,noinsert,fuzzy,preview,noselect'
# ...
```

## :book: Licenza

`rocks-config.nvim` è rilasciato sotto licenza [GPLv3](./LICENSE).

[luarocks-shield]: https://img.shields.io/luarocks/v/neorocks/rocks-config.nvim?logo=lua&color=purple&style=for-the-badge
[luarocks-url]: https://luarocks.org/modules/neorocks/rocks-config.nvim
