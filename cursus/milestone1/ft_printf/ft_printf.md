[[libft]]
#### メモ
make -C オプションで異なる階層の makefileを実行できる。-> libft の利用
put_\*\_fd 関数の出力先のファイルディスクリプタを1 にすれば標準出力 -> いくつか追記すればそのまま出力に流用できる。

- <stdarg.h>
  可変長引数を扱うヘッダ
- va_list
  ```c
  typedef char * va_list;
  ```
  関数の引数は連続したメモリに格納されており、va_listはその中のアドレスを指している。
- va_start
  ```c
  void dummyfunc(char *, ...)
  {
	  va_list ap;
	  va_start(ap, top):
  }
  ```
  va_list型のapが引数の先頭ポインタを指す。
- va_arg
  ```c
  va_list ap;
  int d;
  d = va_arg(ap, int);
  ```
  次の引数を取り出し、int型にキャストして返す。
- va_end
  たぶんだけど内部的に使っているmallocをfreeしている。