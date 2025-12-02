---
created: 2025-11-27
tags:
  - so_long
  - minilibx
aliases:
  - mlx関数チートシート
related:
  - "[[so-long]]"
---

# MiniLibX 関数チートシート

- `mlx_init`
  - Xサーバとの接続を確立する最初の一手。戻り値がNULLなら以降のAPIは利用不可なので即座にエラー処理へ。
  - so_longではここで得た接続IDを `mlx_new_window` や画像ロード系に共有し、終了時に `mlx_destroy_display` まで責任を持って解放する。

- `mlx_new_window`
  - `mlx_init`の接続IDとマップ幅・高さ（タイル数×ピクセル）を渡して描画領域を生成。戻り値NULLならウィンドウ構築前に確保したリソースを解放して脱出。
  - タイトルはレビューで確認されるので `mlx_new_window(mlx, w, h, "so_long")` のように要件どおりの名称をセット。

- `mlx_xpm_file_to_image`
  - `.xpm`スプライトをメモリにロードして画像IDを返す。幅・高さポインタにピクセルサイズが入るので、スプライト解像度のずれを検出可能。
  - 読み込み失敗（NULL）は即時に全画像を破棄→`mlx_destroy_display` まで行ってからエラー終了し、リークや二重freeを避ける。

- `mlx_put_image_to_window`
  - ロード済み画像をウィンドウの `(x * tile_size, y * tile_size)` に貼り付ける。床→差分スプライトの順に描画してタイルの重なりを制御。
  - プレイヤー移動後の再描画や初回マップ描画で大量に呼ばれるため、`set_img` のようなタイル種別→画像ID変換ラッパーを用意するとレビューが通りやすい。

- `mlx_hook`
  - 任意イベント番号＋マスクに自作コールバックを結び付ける汎用API。`2`（KeyPress）で移動入力、`17`（DestroyNotify）でウィンドウ×ボタンを捕捉するのが定番。
  - Linux/X11とmacOSで定数が異なるため、`X11/X.h`のマクロを使うか自前定義を揃えてレビュワーに説明できる状態にしておく。

- `mlx_key_hook`
  - `mlx_hook(win, 2, 0, funct, param)` の糖衣。KeyPress専用で、キーリリースや×ボタンは別フックが必要。簡易実装では `mlx_key_hook(win, deal_key, &game);` だけで済む。

- `mlx_loop`
  - イベント待ち受けの無限ループ。ここに入る前に全フック登録を完了させ、戻らない前提で後続処理を書かない（終了はフック経由で `exit` まで実行）。
  - Busy waitではなくブロッキングなのでCPUを食い潰さない。ループ抜けは `exit` や `mlx_loop_end`（Linux拡張）を併用。

- `mlx_destroy_image`
  - 各画像IDを`NULL`チェック後に破棄。壁バリエーションや床など種類が多いので、`destroy_images` → `destroy_walls` のように分割して二重解放を防ぐ。

- `mlx_destroy_window`
  - ウィンドウIDが生きているときだけ呼び、破棄後は `game->win = NULL` に更新。ESCキーや×ボタン処理で共通の `close_game` 関数から叩くとレビュワーに評価されやすい。

- `mlx_destroy_display`
  - Linux限定の最終クリーンアップでXサーバ接続を閉じる。`mlx_destroy_window` と全画像破棄後に呼び、最後に `free(game->mlx);` まで行うとリークチェッカーが静かになる。

