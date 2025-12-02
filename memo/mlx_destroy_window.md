---
created: 2025-11-27
tags:
  - so_long
aliases:
related:
---
指定されたウィンドウを閉じる関数。

```c
int mlx_destroy_window(void *mlx_ptr, void *win_ptr);
```
# 特徴
- 引数
	- mlx_ptr：セッション接続ID
	- win_ptr：ウィンドウID
- 戻り値
	- 成功時、０
	- 失敗時、実質Void？エラーハンドリングに使うことはまれ



```c
if (game->win)
    {
        mlx_destroy_window(game->mlx, game->win);
        game->win = NULL; // 安全のためNULLにしておく（任意）
    }
```