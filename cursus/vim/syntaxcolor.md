
#### c言語の関数に色を付けたい！
- デフォルトだと関数名がハイライトされない
- syntaxの設定は/usr/share/vim/vim91/syntaxに入っており、言語特有の設定は"言語名.vim"ファイル内で決められている。
- c.vimファイル内ではcFunctionグループを定義し、それがFunction色になるよう設定されている。Function色はIdentifier色のリンクになっており、Identifierはシアン色。
- そのままでは関数ハイライトは適用されない。cFunctionグループが有効にならないからである。
```vimscript
  if exists("c_functions")$
  syn match cFunction "\<\h\w*\ze\_s*("$
  endif
  ```
  .vimrcでc_functionを存在させてやる必要がある。
  ```vimscript
  let c_function = 1
  ```

#### 独自のsyntaxhighlight

[vim日本語ドキュメント](https://vim-jp.org/vimdoc-ja/syntax.html#:syn-files)
runtimepathに設定ファイルを置く。
例えば、cのシンタックスを拡張したい場合は~/.vim/after/syntaxにc.vimを置く。
正規表現でマッチさせてハイライト。
- 注意　なぜか正規表現の'\\b'が有効にならないので'\\<', '\\>'で代用

#### カラースキーム変えたい！
```vimscript
colorsheme habamax
```
/usr/share/vim/vim91/colorに入ってる。:colorsheme space tab で一覧できる

