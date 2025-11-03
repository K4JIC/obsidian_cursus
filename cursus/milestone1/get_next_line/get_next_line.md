
#### プロトタイプ
```c
char *get_next_line(int fd);
```
#### 概要
呼び出すとファイルから一行返す。
もう一度呼び出すと次の行を返す。
readする際のバッファサイズはコンパイル時に決められる。

#### 新概念
##### static variable
関数終了時にリセットされない変数。
過去に呼び出されたときの情報を保持し続ける。

## メモ
- 引数がfdしかないため、過去に読み込んだ情報を取得できない。
  何が困るか。
  たとえば、BUFFER_SIZE = 4 , s = "a \\n bbb \\n cc" を考える。
  gnlを\\n が見つかるまで read して出力する関数として実装すると、
  一回目の呼び出し : "a \\n bb" を read, "a" を return;
  二回目の呼び出し : "b \\n cc" を read, "b" を return;
  二回目の呼び出しで期待される出力は "bbb" であり、"bbb" の出力には一回目の呼び出しの情報が必要になる。引数に頼らず持ち越す必要がある。
  これを実現する方法がstatic variable

- mallocが複雑
  ```c
  save = strjoin(save, buf); //左が更新後のsave 右が更新前のsave
  ```
  strjoin を使って malloc された save 文字列を更新したいが、free のタイミングがない。
  strjoin 関数外で free(save) すると更新後の save 文字列が消えてしまうし、strjoin 関数内でやるとmalloc 以外の第一引数が与えられた場合にややこしい。機能を損なう。
  realloc は制限内での実装が困難。ラッパー関数を使って free(save) を加えるのが現実的か。

- 更新規則
  ```c
  void update(char **new_save, char *old_save);
  
  static char *save = (char *)malloc(1);
  update(&save, save);
  ```
  あむさん (intra : ayhirose) がやってたやつ。
  save を書き換えるために &save を渡して、更新前の save を free するために save を渡している。
- 構造体の利用
  ```c
  typedef struct s_memory
  {
	  char *content;
	  size_t len;
  } t_memory;
  ```
  file内のNULL文字も出力したいから、NULLを終端文字として利用する文字列の考え方は使えない。代わりに生の配列を長さとセットで扱う。メンバlenがポインタ型ではないのでアロー演算子が使えず非常に扱いづらい。
  アロー演算子はメンバの型を問わず使える。構造体自体がポインタ化されていればmemory->len = 0 のようにかける。