
型定義
```c
typedef struct s_memory
{
	unsigned char *content;
	size_t len;
} t_memory;
```

バイナリを表示するために、NULL文字を含む文字列に対応できるようにした。
content : NULL終端していない文字列。文字列の中にNULL文字を含む場合がある。
len          : contentの長さ。

| 変数                          | 説明                                              |
| --------------------------- | ----------------------------------------------- |
| ```static t_memory save;``` | 次のgnl呼び出し時に必要な文字列を保持する。<br>構造体、メンバともにmallocで確保。 |
| ```char *buf;```            | readした文字列を一時的に保持する。<br>mallocで確保。               |
| ```char *line;```           | 出力として返す行。                                       |

| 関数名                           | 機能  |
| ----------------------------- | --- |
| ```char *get_next_line(fd)``` |     |
| ```<br>```                    |     |
