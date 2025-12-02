---
created: 2025-11-25
tags:
  - so_long
aliases:
related:
---
新しいウィンドウを画面上に作成する関数。

[[mlx_init]]でグラフィックサーバとのセッションを確立し、そのセッションの中で新しいTTYやTMUXのペインを開く。

```c
void *mlx_new_window(void *mlx_ptr, int size_x, int size_y, char *title);
```


# 特徴
- 引数
	- mlx_ptr：mlx_initで確立したセッション接続ID
	- size_x, size_y：ウィンドウの横幅と縦幅(マップのマス数×画像のサイズ)
	- title：ウィンドウ上部のバーに表示される文字列
- 戻り値
	- 成功時、Window識別子(ウィンドウを管理する構造体のアドレス)
	- 失敗時、NULL


# コード例
```c
t_game game;

// 1. 接続
game.mlx = mlx_init();
if (game.mlx == NULL) return (ERROR);

// 2. マップサイズからウィンドウサイズを計算
// (例: 10マス × 32px = 320px)
int win_w = game.map_width * 32;
int win_h = game.map_height * 32;

// 3. ウィンドウ作成
game.win = mlx_new_window(game.mlx, win_w, win_h, "so_long");

// 4. エラーチェック（重要！）
if (game.win == NULL)
{
    free(game.mlx); // Linuxなら cleanup が必要
    return (ERROR);
}
```