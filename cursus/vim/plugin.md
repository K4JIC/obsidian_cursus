
#### プラグインマネージャ
vim-plug

#### vim-plug
```vimscript
call plug#begin()$
  Plug 'lambdalisue/vim-fern'$
  Plug 'morhetz/gruvbox'$
call plug#end()$
```
.vimrc内にこのように記述し、:PlugInstall実行でインストールされる
プラグインのカラースキーム設定などの設定を書きたい場合は、end()以降に書く。