---
created: 2025-11-25
tags:
  - so_long
aliases:
related:
---
読み込んだ画像をウィンドウに貼り付ける
[[mlx_destroy_image]]で使い終わった画像メモリを開放する
```c
int mlx_put_image_to_window(void *mlx, void *win, void *img, int x, int y);
```

# 特徴
- 用途
	- マップの描画や、プレイヤーが動いた時の再描画
- 引数
	- mlx：グラフィックサーバとの接続
	- win：ウィンドウの識別ID
	- img：画像メモリのポインタ
	- x, y：ウィンドウ上の座標