---
created: 2025-11-25
tags:
  - so_long
aliases:
related:
---
mlx_key_hookは[[mlx_hook]]のキーボード入力専用・簡易版

```c
int mlx_key_hook(void *win, int (*funct)(), void *param);
```

# 特徴
- 引数
	- win：ウィンドウの識別ID
	- funct：キーが押されたときに呼び出す自作関数
	- param：functの引数
- mlx_hookとの違い
	- mlx_key_hookの中身は単純にmlx_hookを呼び出しているだけ
	- ```c
	  // mlx_key_hook を使うと...
	  mlx_key_hook(win, my_func, param);
	  // ↓ 実は裏でこれをやっているのと同じ！
	  mlx_hook(win, 2, 0, my_func, param); // 2 = KeyPress
	  ```
	- メリット
		- イベント番号やマスクを書かなくていい
	- デメリット
		- 「キーを押したとき」しか反応できない。「キーを話したとき」や「❌ボタンを押したとき」には反応できない

# コード例
```c
int deal_key(int keycode, t_game *game)
{
    printf("押されたキーの番号: %d\n", keycode);
    
    // ESCキー(53)なら終了
    if (keycode == 53)
        exit(0);
        
    return (0);
}

int main(void)
{
    // ... 初期化処理 ...

    // 2. フックを登録する
    // 「winでキーが押されたら deal_key を実行せよ。その際 &game を渡せ」
    mlx_key_hook(win, deal_key, &game);
    mlx_loop(mlx);
}
```