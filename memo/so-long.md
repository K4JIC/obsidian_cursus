---
created: 2025-11-19
tags:
  - so_long
aliases:
related:
---
# 問題の要約
- 2Dマップをもとに宝取りゲームを作成する。
- ゲーム作成にはMiniLibXを使用する。
- Mapには必ず1つの出口、1つの開始位置、1つ以上の回収品が含まれている必要がある。



# 必要知識の洗い出し
- MiniLibXの操作
	- ライブラリの連携
		- Makefileで.aファイルをコンパイル時にリンクする方法
	- ウィンドウ管理
		- 接続の初期化：[[mlx_init]]
		- ウィンドウの作成：[[mlx_new_window]]
	- 画像処理
		- .xpm画像ファイルを読み込んでメモリにロードする：[[mlx_xpm_file_to_image]]
		- ロードした画像をウィンドウの特定座標に貼り付ける：[[mlx_put_image_to_window]]
	- イベント駆動プログラム
		- 従来のmain関数で上から下に流れる処理とは異なり、「キーが押されたら○○する」という待ち受け状態を作る概念
		- キーボード入力の待ち受け：[[mlx_hook]]または　[[mlx_key_hook]]
		- イベントを待ち受ける無限ループ：[[mlx_loop]]
	- 後始末関数
		- 使い終わった画像メモリを解放する：[[mlx_destory_image]]
		- ウィンドウを閉じて関連リソースを破棄する：[[mlx_destroy_window]]
		- グラフィックサーバとの接続を切断し、サーバ側のリソースを解放する：[[mlx_destory_display]]
- 構造体でのゲーム情報設計
	- ゲームの変数は多くなるため、ばらばらの変数ではなく、1つの巨大な構造体にまとめる必要がある
	- 構造体の入れ子：例t_gameの中に、t_mapやt_imgを持たせるなど
	- ポインタ渡しで、関数間で構造体ポインタを回して、状態を更新する
- マップ解析におけるファイル操作とメモリ管理
	- GNLを使用して、マップを一行ずつ読み込む。
	- 読み込んだマップをchar \*\*mapのような形でメモリに格納する
- アルゴリズム
	- Flood Fillアルゴリズムで、初期位置からすべてのアイテムと出口に到達できるかをチェックする。


## 課題要件
- Game
	- プレイヤーの目的はマップ上の全ての収集物を集め、可能な限り最短のルートで脱出すること
	- 移動にはW, A, S, Dキーを使用する
	- プレイヤーは上下左右の4方向に移動できる必要がある
	- プレイヤーは壁の中に入ることはできない
	- 移動するたびに、現在の移動回数がシェルに表示される
	- リアルタイムである必要はない
- Graphic management
	- ウィンドウ操作はスムーズに実行される必要がある(別のウィンドウへの切り替え、最小化など)
	- ESCキーを押すと、ウィンドウが閉じられ、プログラムが正常に終了する
	- ウィンドウの❌ボタンをクリックすると、ウィンドウが閉じられ、プログラムが正常に終了する
	- ピクセル描画ではなく、画像描画する
- Map
	- マップには「壁」、「収集物」、「空きスペース」の3つの要素で構築される
	- マップは次の5つの文字で構成できる
		- 0：空きスペース→草むら
		- 1：壁→木や柵
		- C：収集物→コイン
		- E：マップの出口→アモアス
		- P：プレイヤーの開始位置→ネコ
	- マップには必ず、出口が1つ、開始位置が1つ、収集物が1つ含まれている必要がある
	- PとEが複数ある場合は、エラーメッセージを表示する
	- マップは長方形である必要がある
	- マップが壁に囲まれていなければエラーを返す
	- マップ内に有効なパスがあるかどうかチェックする必要がある
	- ファイル内で設定ミスが見つかった場合、プログラムは正常終了し、「Error\n」に続いて任意のエラーメッセージを明示的に返す必要がある

# 解法の検討
- とりあえずMiniLibXで遊んでみる
- map_generaterを作成する(課題の要件的には不要)
- マップ読み込みを作る
	- ファイルを最後まで読み込んでchar \*\*mapに格納する
- 読み込んだマップに合わせて画像をウィンドウに並べる
- キー入力でプレイヤーの座標を書き換え、画像を再描画できるようにする
	- キー入力を受け付ける
	- キー入力によってプレイヤーの座標を変更する
	- プレイヤーが元いた座標は床に変更する
	- プレイヤーの座標が収集物の座標と重なったら1ポイントアップ
	- ポイントが収集物の数を一致したらExitを有効化する
	- ポイントが収集物の数と一致しないうちはExitを消してはいけない
	- ポイントが十分かつExitに触れたらゲームクリア
- map_validate_checkerを作成する
	- ファイルチェック
		- 拡張子は.berか
		- ファイルは開けるか？
		- ファイルが空ではないか？
	- 形状チェック
		- 全ての行の長さが同じであること
		- 長方形であること
	- 壁で囲まれていること
	- 不正な文字が含まれていないこと
	- 構成要素の数のチェック
	- 有効なパスがあること



# 関数・変数設計
疑似コード例
  main:
    arr1 = 入力を読む
    arr2 = 入力を読む
    set1 = remove_duplicates(arr1)
    set2 = remove_duplicates(arr2)
    result = intersect_arrays(set1, set2)
    出力する
## マップ読み込み
```c
/*
GNLで一行読む
読んだ行をt_listに追加する
リストの個数をカウントする
カウント分だけmallocしてchar **mapを作る
リストの中身をmapにコピーする
使い終わったリストを解放する
*/
char	**read_map(char *filepath)
{
	int		fd;
	char	*raw_line;
	char	*trimmed_line;
	t_list	*head;

	head = NULL;
	fd = open(filepath, O_RDONLY);
	while (1)
	{
		raw_line = ft_gnl(fd);
		if (raw_line == NULL)
			break ;
		trimmed_line = ft_strtrim(raw_line, "\n");
		free(raw_line);
		if (trimmed_line == NULL)
		{
			ft_lstclear(&head, free);
			close(fd);
			return (NULL);
		}
		ft_lstadd_back(&head, ft_lstnew(trimmed_line));
	}
	close(fd);
	return (lst_to_arr(head));
}

static char	**lst_to_arr(t_list *head)
{
	int	size;
	int	i;
	char	**map;
	t_list	*tmp;
	t_list	*next_node;

	size = ft_lstsize(head);
	map = malloc(sizeof(t_list) * (size + 1));
	if (map == NULL)
	{
		ft_lstclear(&head, free);
		return (NULL);
	}
	i = 0;
	tmp = head;
	while (tmp != NULL)
	{
		map[i] = tmp->content;
		next_node = tmp->next;
		free(tmp);
		tmp = next_node;
		i++;
	}
	map[i] = NULL;
	return (map);
}
```
## マップに合わせて画像を表示
```c
/*
1.ウィンドウの初期化
2.ウィンドウの立ち上げ
3.画像の読み込み
4.画像の書き出し
1~3をmainで4は独立させる
*/

int main(int ac, char **av)
{
	if (ac != 2)
	{
		write(2, "Usage: ./so_long <map_filepath>\n", 33);
		return (EXIT_FAILURE);
	}
	if (start_game(av[1]) == EXIT_FAILURE)
	{
		write(2, "", );
		return (EXIT_FAILURE)
	}
	return (EXIT_SUCCESS);
}

//こっから下は別ファイルにする？
typedef struct s_texture
{
	void *wall;
	void *floor;
	void *collectible;
	void *player;
	void *exit;
} t_texture;

typedef struct s_game
{
	void *mlx;
	void *win;
	char **map;
	int map_w;
	int map_h;
	t_textures img;
} t_game;

int start_game(char *filepath)
{
	t_game game;
	t_textures texture;
	
	if (set_map_info(filepath, &game) == EXIT_FAILURE)
		return (EXIT_FAILURE);
	if (launch_window(&game) == EXIT_FAILURE)
		return (EXIT_FAILURE);
	if (map_image_output(&game) == EXIT_FAILURE)
		return (EXIT_FAILURE);
	mlx_loop(game.mlx);
	return (EXIT_SUCCESS);
}

int set_map_info(char *filepath, t_game *game)
{
	game->map = read_map(filepath);
	if (game->map == NULL)
	{
		write(2, "", );
		return (EXIT_FAILURE);
	}
	return (EXIT_SUCCESS);
}

int launch_window(t_game *game)
{
	game->mlx = mlx_init();
	if (game.mlx == NULL)
	{
		write(2, "", );
		return (EXIT_FAILURE);
	}
	init_window_size(game);
	game->win = mlx_new_window(game.mlx, game.map_w, game.map_h, "so_long");
	if (game->win == NULL)
	{
		write(2, "", );
		return (EXIT_FAILURE);
	}
	return (EIXT_SUCCESS);
}

void init_window_size(t_game *game)
{
	int height;
	
	game->map_w = ft_strlen(game->map[0]);
	height = 0;
	while (game->map[y])
		height++;
	game->map_h = height;
}

int map_image_output(t_game *game)
{
	if (init_textures(game->mlx, &game->img) == EXIT_FAILURE)
		return (EXIT_FAILURE);
	if (draw_map(game) == EXIT_FAILURE)
		return (EXIT_FAILURE);
	return (EXIT_SUCCESS);
}

int init_textures(void *mlx, t_texture *img)
{
	int w;
	int h;

	img->wall = mlx_xpm_file_to_image(mlx, WALL, &w, &h);
	img->floor = mlx_xpm_file_to_image(mlx, FLOOR, &w, &h);
	img->collectible = mlx_xpm_file_to_image(mlx, COLLECTIBLE, &w, &h);
	img->player = mlx_xpm_file_to_image(mlx, PLAYER, &w, &h);
	img->exit =  mlx_xpm_file_to_image(mlx, EXIT, &w, &h);
	if (!img->wall || !img->floor || !img->collectible
		|| !img->player || !img->exit)
	{
		write(2, "", );
		return (EXIT_FAILURE);
	}
	return (EXIT_SUCCESS);
}

int	draw_map(t_game *game)
{
	int		x;
	int		y;
	void	*img;

	y = 0;
	while (game->map[y])
	{
		x = 0;
		while (game->map[y][x])
		{
			mlx_put_image_to_window(game->mlx,
				game->win, game->img.floor, x * 32, y * 32);
			img = NULL;
			set_img(game, &img, x, y);
			if (img != NULL && img != game->img.floor)
				mlx_put_image_to_window(game->mlx,
					game->win, img, x * 32, y * 32);
			x++;
		}
		y++;
	}
	return (EXIT_SUCCESS);
}

　
//分割しないといけないかも
int set_img(t_game *game, void **img, int x, int y)
{
	int w;
	int h;
	char tile;
	
	w = game->map_w;
	h = game->map_h;
	tile = game->map[y][x];
	
	if (tile == '0')
		*img = game->img.floor;
	else if (tile == 'C')
		*img = game->img.collectible;
	else if (tile == 'P')
		*img = game->img.player;
	else if (tile == 'E')
		*img = game->img.exit;
	else if (tile == '1')
	{
		if (x == 0 && y == 0)
            *img = game->img.wall_lu;
        else if (x == w - 1 && y == 0)
            *img = game->img.wall_ru;
        else if (x == 0 && y == h - 1)
            *img = game->img.wall_ll;
        else if (x == w - 1 && y == h - 1)
            *img = game->img.wall__rl;
        else if (y == 0 || y == h - 1)
            *img = game->img.wall_h;
        else
            *img = game->img.wall_v;)
	}
}
```
## キー入力でプレイヤーを操作
```c
/*
	- キー入力を受け付ける
	- キー入力によってプレイヤーの座標を変更する
	- プレイヤーが元いた座標は床に変更する
	- プレイヤーの座標が収集物の座標と重なったら1ポイントアップ
	- ポイントが収集物の数を一致したらExitを有効化する
	- ポイントが収集物の数と一致しないうちはExitを消してはいけない
	- ポイントが十分かつExitに触れたらゲームクリア
*/
int start_game(char *filepath)
{
	//省略。mlx_loopの前にmlx_hookを置く
	mlx_hook(game.win, KEY_PRESS, KEY_PRESS_MASK, key_press, &game)://KEY_PRESS, MASKはヘッダーファイルで定義
	mlx_hook(game.win, CROSS_BUTTON, CROSS_BUTTON_MASK, close_game, &game);//CROSS_BUTTON, MASKはヘッダーファイルで定義
	mlx_loop(game.mlx);
	return (EXIT_SUCCESS);
}

int key_press(int keycoke, t_game *game)
{
	//t_gameにプレイヤーの座標と収集物の数、移動した数も追加する
	if (keycode == KEY_ESC)
		close_game(game);
	if (keycode == KEY_W)
		move__player(game, 0, -1);
	else if (keycode == KEY_A)
		move_player(game, -1, 0);
	else if (keycode == KEY_S)
		move_player(game, 0, 1);
	else if (keycode == KEY_D)
		move_player(game, 1, 0);
	return (0);
}

void move_player(t_game *game, int x, int y)
{
	int next_x;
	int next_y;
	char next_tile;
	
	next_x = game->player_x + x;
	next_y = game->player_y + y;
	next_tile = game->map[next_y][next_x];
	if (is_wall(game, next_x, next_y))
		return ;
	if (check_exit(game, next_x, next_y))
		return ;
	check_collectible(game, next_x, next_y);
	update_coordinate(game, next_x, next_y);
}

int is_wall(t_game *game, int x, int y)
{
	return (game->map[y][x] == '1');
}

int check_exit(t_game *game, int x, int y)
{
	if (game->map[y][x] == 'E')
	{
		if (game->collectible_count == 0)
		{
			ft_putstr_fd("Game Clear!! \n", 2);
			close_game(game);
		}
		return (1);
	}
	return (0);
}

void check_collectible(t_game *game, int x, int y)
{
	if (game->map[y][x] == 'C')
	{
		game->collectible_count--;
		game->map[y][x] = '0';
	}
}

void update_coordinate(t_game *game, int x, int y)
{
	game->map[game->player_y][game->player_x] = '0';
	game->map[y][x] = 'P';
	game->player_x = x;
	game->player_y = y;
	game->move_count++;
	ft_putstr_fd("Moves: ", 1);
	ft_putnbr_fd(game->move_count, 1);
	ft_putstr_fd("\n", 1);
	draw_map(game);
}

//set_map関数の中に入れ込む
void init_game_vars(t_game *game)
{
	int x;
	int y;
	
	y = 0;
	while (game->map[y])
	{
		x = 0;
		while (game->map[y][x])
		{
			if (game->map[y][x] == 'P')
			{
				game->player_x = x;
				game->player_y = y;
			}
			else if (game->map[y][x] == 'C')
				game->collectible_count++;
			x++;
		}
		y++;
	}
}
```
## ゲームを終了する
```c
int close_game(t_game *game)
{
	destory_images(game);
	destory_window(game);
	destory_display(game);
	free_map(game->map);
	exit(EXIT_SUCCESS);
	return (0);
}

void free_map(char **map)
{
	int i;
	
	if (map == NULL)
		return ;
	i = 0;
	while (map[i])
	{
		free(map[i]);
		i++;
	}
	free(map);
}

void destroy_images(t_game *game)
{
	destroy_walls(game);
	if (game->img.floor)
		mlx_destory_image(game->mlx, game->img.floor);
	if (game->img.collectible)
		mlx_destory_image(game->mlx, game->img.collectible);
	if (game->img.player)
		mlx_destory_image(game->mlx, game->img.player);
	if (game->img.exit)
		mlx_destory_image(game->mlx, game->img.exit);
}

void destroy_walls(t_game *game)
{
	if (game->img.wall_lu)
		mlx_destory_image(game->mlx, game->img.wall_lu);
	if (game->img.wall_ru)
		mlx_destory_image(game->mlx, game->img.wall_ru);
	if (game->img.wall_ll)
		mlx_destory_image(game->mlx, game->img.wall_ll);
	if (game->img.wall_rl)
		mlx_destory_image(game->mlx, game->img.wall_rl);
	if (game->img.wall_v)
		mlx_destory_image(game->mlx, game->img.wall_v);
	if (game->img.wall_h)
		mlx_destory_image(game->mlx, game->img.wall_h);
}

void destroy_window(t_game *game)
{
	if (game->win)
		mlx_destroy_window(game->mlx, game->win)
}

void destroy_display(t_game *game)
{
	if (game->mlx)
	{
		mlx_destroy_display(game->mlx);
		free(game->mlx);
	}
}
```
## マップを検査する
```c
/*
	- ファイルチェック//これはmainから呼び出す
		- 拡張子は.berか
		- ファイルは開けるか？
		- ファイルが空ではないか？
	- 形状チェック
		- 全ての行の長さが同じであること
		- 長方形であること//正方形は長方形に含まれる
	- 壁で囲まれていること
	- 不正な文字が含まれていないこと
	- 構成要素の数のチェック
	- 有効なパスがあること
*/
int validate_map(t_game *game)
{
	if (check_rectangular(game) == EXIT_FAILUER)
		return (EXIT_FAILURE);
	if (check_wall(game) == EXIT_FAILUER)
		return (EXIT_FAILURE);
	if (check_chars(game) == EXIT_FAILUER)
		return (EXIT_FAILURE);
	if (check_path(game) == EXIT_FAILUER)
		return (EXIT_FAILURE);
	return (EXIT_SUCCESS);
}

//この関数はmain関数の中から呼び出す
int check_input(char *filepath)
{
	size_t len;
	int fd;
	
	if (filepath == NULL)
		return (EXIT_FAILURE);
	len = ft_strlen(filepath);
	if (len < ft_strlen(MAP_EXTENSION))
	{
		ft_putstr_fd("Error: The file name must be in the format <filepath/filename>.ber\n", 2)
		return (EXIT_FAILURE);
	}
	fd = open(filepath, O_RDONLY)
	if (fd < 0)
	{
		ft_putstr_fd("Error: Not found or Permission denied.\n", 2);
		return (EXIT_FAILURE);
	}
	close(fd);
	return (EXIT_SUCCESS);
}

int check_rectangular(t_game *game)
{
	int i;
	size_t width;
	
	if (!game->map || !game->map[0])
		return (EXIT_FAILURE);
	width = ft_strlen(game->map[0]);
	game->map_w = (int)width;
	i = 1;
	while (game->map[i])
	{
		if (ft_strlen(game->map[i]) != width)
		{
			ft_putstr_fd("Error: Map is not rectangular.\n", 2);
			return (EXIT_FAILURE);
		}
		i++;
	}
	game->map_h = i;
	return (EXIT_SUCCESS);
}

int check_wall(t_game *game)
{
	int i;
	
	i = 0;
	while (i < game->map_w)
	{
		if (game->map[0][i] != '1'
			|| game->map[game->map_h - 1][i] != '1')
		{
			ft_putstr_fd("Error: Map is not surrounded by wall.\n", 2);
			return (EXIT_FAILURE);
		}
		i++;
	} 
	i = 0;
	while (i < game->map_h)
	{
		if (game->map[i][0] != '1'
			|| game->map[i][game->map_w - 1] != '1')
		{
			ft_putstr_fd("Error: Map is not surrounded by wall.\n", 2);
			return (EXIT_FAILURE);
		}
		i++;
	}
	return (EXIT_SUCCESS);
}

int check_chars(t_game *game)
{
	int p_count;
	int e_count;
	int c_count;
	
	p_count = 0;
	e_count = 0;
	c_count = 0;
	count_chars(game, &p_count, &e_count, &c_count);
	if (p_count != 1)
	{
		ft_putstr_fd("Error: Map must have exactly one Start(P).\n", 2);
		return (EXIT_FAILURE);
	}
	if (e_count != 1)
	{
		ft_putstr_fd("Error: Map must have exactly one Exit(E).\n", 2);
		return (EXIT_FAILURE);
	}
	if (c_count < 1)
	{
		ft_putstr_fd("Error: Map must have at least one Collectible(C).\n", 2);
		return (EXIT_FAILURE);
	}
	game->collectible_count = c_count;
	return (EXIT_SUCCESS);
}

void count_chars(t_game *game, int *p_count, int *e_count, int *c_count)
{
	char c;
	int x;
	int y;

	y = 0;
	while (game->map[y])
	{
		x = 0; 
		while (game->map[y][x])
		{
			c = game->map[y][x];
			if (c != '0' && c != '1' && c != 'C' && c != 'E' && c != 'P')
			{
				ft_putstr_fd("Error: Map contains invalid characters.\n", 2);
					return (EXIT_FAILURE);
			}
			if (c == 'P')
				*p_count++;
			else if (c == 'E')
				*e_count++;
			else if (c == 'C')
				*c_count++;
			x++;
			}
		y++;
	}
}

int check_path(t_game *game)
{
    char    **tmp_map;
    int     y;
    int     x;

    tmp_map = copy_map(game);
    if (tmp_map　== NULL)
        return (EXIT_FAILURE);
    flood_fill(tmp_map, game->player_x, game->player_y);
    y = 0;
    while (tmp_map[y])
    {
        x = 0;
        while (tmp_map[y][x])
        {
            if (tmp_map[y][x] == 'C' || tmp_map[y][x] == 'E')
            {
                free_map(tmp_map);
                ft_putstr_fd("Error: Map path is invalid (Unreachable).\n", 2);
                return (EXIT_FAILURE);
            }
            x++;
        }
        y++;
    }
    free_map(temp_map);
    return (EXIT_SUCCESS);
}

char **copy_map(t_game *game)
{
    char    **copy;
    int     i;

    copy = malloc(sizeof(char *) * (game->map_h + 1));
    if (copy == NULL)
        return (NULL);

    i = 0;
    while (i < game->map_h)
    {
        copy[i] = ft_strdup(game->map[i]);
        if (!copy[i])
        {
            free_map(copy); 
            return (NULL);
        }
        i++;
    }
    copy[i] = NULL;
    return (copy);
}

void flood_fill(char **map, int x, int y)
{
    if (map[y][x] == '1' || map[y][x] == 'F')
        return ;

    map[y][x] = 'F';
    flood_fill(map, x + 1, y);
    flood_fill(map, x - 1, y);
    flood_fill(map, x, y + 1);
    flood_fill(map, x, y - 1);
}
```

# 抽象化
### 1. イベント駆動アーキテクチャ (Event-Driven Architecture)
- **構造:** `mlx_loop` で無限ループし、キー入力等のイベントを待機する非同期処理。
- **抽象化:** イベントループとコールバックの理解。
- **応用:**
    - **Webサーバー (Nginx/Node.js):** リクエスト待機 → 処理 → レスポンスの流れ。
    - **GUIアプリ:** Windows/macOSアプリ開発の基礎。
    - **次:** `minishell` (プロンプト待機), `webserv` (HTTPリクエスト待機) に直結。

### 2. リソースのライフサイクル管理 (Resource Lifecycle)
- **構造:** `init` で確保 → `close` で破棄。エラー時のロールバック (途中まで確保した分の解放) を徹底。
- **抽象化:** **RAII** (Resource Acquisition Is Initialization) と **Graceful Shutdown**。
- **応用:**
    - **バックエンド:** DB接続、ファイルハンドル、ソケットの開閉管理。リーク＝サーバー死。
    - **インフラ:** 構築 (Provisioning) と撤去 (Decommissioning) の順序依存性。

### 3. コンテキストの伝播 (Context Propagation)
- **構造:** 巨大な構造体 `t_game` をポインタで各関数に回す。
- **抽象化:** **状態管理 (State Management)** と **依存性の注入 (DI)** の原型。
- **応用:**
    - **Web開発:** リクエストスコープ (ユーザー情報やDBセッション) の持ち回り。
    - **大規模開発:** グローバル変数を避け、疎結合なコードにするための基本パターン。

### 4. バリデーションと "Fail Fast"
- **構造:** 読み込み時に拡張子、壁、経路 (Flood Fill) を厳密にチェックし、不正なら即 `Error`。
- **抽象化:** **入力検証 (Input Validation)** と **Fail Fast** (早く失敗させる) 思想。
- **応用:**
    - **セキュリティ:** 外部入力は「悪意がある」前提でサニタイズ。
    - **堅牢性:** エラーを早期発見し、デバッグと運用を楽にする。

### 5. アルゴリズムの実践 (Flood Fill)
- **構造:** 再帰関数によるマップ探索・塗りつぶし。
- **抽象化:** **グラフ探索 (DFS/BFS)**。
- **応用:**
    - **経路探索:** カーナビ、配送ルート。
    - **関係性解析:** SNS等のグラフデータ解析。
    - **次:** `cub3d` やアルゴリズム課題での再帰的思考。




# 学びと反省
- マップは描画されるようになった
- しかし、何も表示されていないエリアがあったり、grass_floorの重ね合わせができていない。
	- 何も表示されない問題は、プレイヤー画像のサイズが32ぴくせるじゃなかったからでした。
	  32ピクセルにしたら表示されるようになりました。
- プレイヤーの画像も表示されない
	- 解決
- 動くものが作れた。あとは画像の重ね合わせとマップの有効判定。
	- マップの有効判定は完了→テストはまだ
	- 画像の重ね合わせはMLXではアルファチャンネルの処理ができないらしく、自分で処理する必要があるとのこと。
		- 結局画像の重ね合わせは諦め、一体化した画像を作成して代用した。
- 2階だけCCのバージョンが異なり、3,4階では通ったコンパイルが2階だと通らないということがあった
- どこでメモリを確保していて、どこでフリーしているのかを意識する必要がある。
- レビューなんとか突破。しかし一人目のレビューではだいぶ甘く見てもらったように感じているので、自分の甘いところ、特にメモリの管理は今後意識していかなればいけない。
  Milestone2が終わったら一度しっかり設計について勉強したい。

# 気をつけるポイント
- **実行ファイルの名前とコンパイル**
	- `make` コマンドで問題なくコンパイルできること（リリンクしてはいけません）。
	- 生成される実行ファイルの名前が正確に `so_long` であること。
    

- **マップ読み込み (Map reading)**
	- 異なる種類のマップ（正しいマップ、不正なマップ）でテストすること。
	- 異なる行サイズのマップでテストすること（長方形かどうかのチェックなど）。
	- **【重要】エラー処理:** 設定ファイル（マップ）が不正な場合、プログラムは「Error」を返して、正しく終了すること。
	    - 例: 未知の文字がある、キー（P, E, Cなど）が重複している、ファイルのパスが無効、など。
    - 不正な場合にクラッシュ（Segfaultなど）してはいけません。

- **表示の技術的要素 (Technical elements of the display)**
	- プログラム起動時にウィンドウが開くこと。
	- 実行中、ウィンドウが開いたまま維持されること。
	- **ウィンドウの最小化・隠蔽:** 別のウィンドウで隠したり、最小化してから元に戻したりしても、ウィンドウの中身（描画内容）が壊れず、一貫性を保っていること。

- **ユーザーイベント処理 (User basic events)**
  このセクションで1つでも失敗すると、**その時点で0点**になります。
	- **「閉じる」ボタン:** ウィンドウ左上の赤い「×」ボタンをクリックすると、ウィンドウが閉じ、プログラムが**きれいに終了**すること（メモリリーク等なし）。
	- **ESCキー:** `ESC` キーを押すと、ウィンドウが閉じ、プログラムが**きれいに終了**すること。
	    - ※他のキー（例: 'Q'）で終了できても良いが、ESCは必須。
	- **矢印キー（またはWASD）:** 矢印キー（またはWASD/ZQSD）を押した際、キー入力が正しく認識されていること。

---

- **動作と挙動 (Movements)**
	- **初期位置:** プレイヤーがマップファイル（`.ber`）で指定された正しい位置（'P'の場所）に出現すること。    
	- **移動:** 矢印キーでマップ上の全方向に移動できること。
	- **プレイアビリティ:** ゲームとしてちゃんと遊べる状態であること。

---

- **壁とスプライト (Walls & Sprites)**
  ここがゲームロジックの核心です。
	- **壁の判定:** 壁（'1'）の画像が正しく配置され、プレイヤーは**壁を通り抜けてはいけない**。
	- **アイテム回収:** アイテム（'C'）の画像が正しく配置され、プレイヤーがその上を歩くと**回収（消える）**こと。
	- **ゴール判定:** 出口（'E'）の画像が正しく配置されていること。プレイヤーは**「全てのアイテムを回収した後」**でないと、出口に入ってクリアすることができない。
	    - ※アイテムが残っている状態で出口に乗っても、ゲームが終了してはいけません。
	- **プレイヤー:** プレイヤーの画像が正しく配置され、壁以外の場所なら全方向に動けること。
    

---

- **カウンター (Counter)**
	- プレイヤーが移動した回数をカウントし、その数値を**シェル（ターミナル画面）に表示**すること。
    - ※ゲーム画面内への表示は必須ではありません（ボーナス扱いの場合が多いですが、必須要件は「シェルへの表示」です）。

- **画像の使用方法 (Image usage)**
	- **`mlx_put_image_to_window` を使用しているか？**
    - 画像をウィンドウに貼り付ける関数を使っているかが評価されます。
    - **NG例:** `mlx_put_pixel` などを使って、1ドットずつ描画するのは禁止です。