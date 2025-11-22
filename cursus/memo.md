
- norminette はver. 3.3.55 を使う。
- キーボード設定は/etc/default/keyboard
- バイナリエディタはod
- バイナリファイルをテキストエディタで開くと、プログラム本文で""で囲って書いた部分が文字のまま表示される。
- ""で囲った内容に重複がある場合は一つだけ表示される。
- login : root でルートユーザーでログイン
- ctrl + D でログアウト
- https://www.server-world.info/query?os=Debian_10&p=password



#### AppArmor
debian系統に搭載されているセキュリティソフト。名前ベースのアクセス制御で、LSM（Linux Security Module）を用いて実装されている仕組み。
LSMとはLinuxカーネルのフレームワークで、シェルより低レイヤで動いてアクセス制御するシステム。
（/etc/apparmor.d/アプリ名）AppArmorはアプリごとにユーザーのアクセス権限を設定する。これはファイル所有者や管理者が付与するパーミッションよりも優先され、管理者すら読み込み不可なファイルを生成できる。これを、パーミッション設定などの「任意アクセス制御（DAC : discretionary access control）」と比較して「強制アクセス制御（MAC : Mandatory Access Control）」という。
同様のMAC機能を提供するセキュリティソフトとしてSELinuxがある。こちらはrhel, fedra, rockyディストリなどに標準搭載されており、ファイル名だけを参照するAAと異なり、ファイル形式やメタデータも参照してアクセスを制御する。

使用方法も知っておくとなおよし

https://gihyo.jp/admin/serial/01/ubuntu-recipe/0798


#### UFW(Uncomplicated Firewall)
iptables : linuxのパケットフィルタリング機能（netfilter）を直接操作するコマンド
ufw : iptablesをかんたんに使うためのフロントエンド

一般的なサーバー・デスクトップ
→ **ufw が便利で安全**  
（SSH、HTTP、HTTPS など基本設定は簡単）
特殊なネットワーク設定が必要な場合
→ **iptables の直接操作が必要**  
（NAT・ポートフォワード・高度なマッチ条件など）


#### NAT,  ポートフォワーディング
- **NAT (Network Address Translation)**
  IPアドレスを変換する技術。プライベートIPアドレス↔グローバルIPアドレス。
  グローバルIPアドレスをすべての機器に付与していたら枯渇するので、節約のためにプライベートIPアドレスが必要。
  LANからインターネットにつながるときプライベートからグローバルの変換が必要で、それを実現するのがNAT。
- **NATの弊害**
  プライベート->グローバルの変換はできるが、グローバル->プライベートの変換をする際に、どの機器に向けたパケットかわからないので変換できない。
- **ポートフォワーディング**
  インターネットから特定のポート番号宛に通信が届いたとき、特定の機器にパケットを転送する機能。
  たとえば、宛先ポートが80番のパケットはプライベートアドレス192.168.1.1の機器に転送するなど。
##### 仮想マシンにおけるNAT, ブリッジ接続

GUIの排除
https://docs.redhat.com/ja/documentation/red_hat_enterprise_linux/7/html/system_administrators_guide/sect-managing_services_with_systemd-targets

pts,tty