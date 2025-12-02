---
created: 2025-11-27
tags:
  - so_long
aliases:
related:
---
X Serverとの接続を切断する関数

```c
int mlx_destroy_display(void *mlx_ptr);
```

# 特徴
- 引数
	- mlx_ptr：セッション接続ID
- 戻り値
	- 成功時、０
	- 失敗時の戻り地のチェックは不要、実質void