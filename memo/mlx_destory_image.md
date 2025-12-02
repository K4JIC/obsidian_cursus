---
created: 2025-11-27
tags:
  - so_long
aliases:
related:
---
読み込んだ画像メモリを解放する関数。

```c
int mlx_destroy_image(void *mlx_ptr, void *img_ptr);
```

# 特徴
- 引数
	- mlx_ptr：セッション接続ID
	- img_ptr：画像ID
- 戻り値
	- 成功時は、０
	- 失敗時は、非０


# コード例
```c
画像IDのNULLチェックを必ず行う
if (game->img.floor)
	mlx_destroy_image(game->mlx, game->img.floor);
```