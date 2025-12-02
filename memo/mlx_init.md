---
created: 2025-11-25
tags:
  - so_long
aliases:
related:
---
mlx_initはコネクション確立（3ウェイハンドシェイク）

[[socket]]()と[[connect]]()を同時に行うようなもの。
```c
void *mlx_init(void);
```
# 特徴
- 引数：なし
- 戻り値
	- 成功したとき、接続ID（確立されたセッション情報、画面の設定など）
	- 失敗したとき、NULL


# コード例
```c
#include "mlx.h"
#include <stdlib.h> // exit用

int main(void)
{
    void *mlx;

    mlx = mlx_init();
    if (mlx == NULL)
    {
        // 接続失敗（メモリ不足 or X Serverが見つからない）
        // エラーメッセージを出して終了
        write(2, "Error\nMinilibX init failed\n", 25);
        return (1);
    }

    // ... 成功したら続きの処理 ...
}
```