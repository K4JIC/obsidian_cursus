---
created: 2025-11-25
tags:
  - so_long
aliases:
related:
---
.xpmファイル(画像)を読み込んで、プログラム内で使える形に変換する
```c
void *mlx_xpm_file_to_image(void *mlx, char *filename, int *width, int *height);
```

# 特徴
- 用途
	- ゲーム開始時に、壁や床、プレイヤーなどの画像を読み込むときに使う。
- 引数
	- mlx：グラフィックサーバとの接続
	- filename：画像ファイルのパス
	- width：画像のサイズを格納するための変数のアドレス
	- height：画像のサイズを格納するための変数のアドレス
- 戻り値
	- 成功すると、画像のID(ポインタ)
	- 失敗すると、NULL