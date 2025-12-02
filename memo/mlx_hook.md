---
created: 2025-11-25
tags:
  - so_long
aliases:
related:
---
プレイヤーの操作を受け付けるための関数
特定のイベント(キーが押された、❌ボタンが押されたなど)が発生したときに、自作の関数を呼び出す
```c
int mlx_hook(void *win, int x_event, int x_mask, int (*funct)(), void *param);
```

# 特徴
- 用途
	- ON_KEYDOWN：プレイヤー移動
	- ON_DESTORY：ウィンドウを閉じる
- 注意点
	- Linux(X11)とMacではイベント番号が違うことがあるため、X11/X.hをインクルードして、定数を使うのが一般的。（自分で定義する？？）
	- ２（KeyPress）：キーが押されたとき
	- 17（DestroyNotify）：❌ボタンが押されたとき
- 引数
	- win：ウィンドウの識別ID
	- x_event：イベントの種類を番号で指定する(番号は自分で指定？)
	- x_mask：どのイベント入力を許可するかのビットマスク
	- \*funct：イベントが発生した際に実行させたい自作関数
	- param：functに渡したい引数



```c
// 自作の関数（スクリプト）
// 引数の param には、hook登録時に渡した &game が入ってくる
int key_press(int keycode, t_game *game)
{
    if (keycode == ESC)
        exit_game(game);
    move_player(game, keycode);
    return (0);
}

int main(void)
{
    t_game  game; // 全設定が入った構造体

    // ... initやwindow作成 ...

    // フックの設定（監視ルールの追加）
    // ウィンドウ(win)で、キー押し(2)があったら、
    // マスク(1L<<0)を通して、key_pressを実行せよ。
    // その時、&game を引数として渡せ。
    mlx_hook(game.win, 2, 1L<<0, key_press, &game);

    mlx_loop(game.mlx); // 監視開始
}
```