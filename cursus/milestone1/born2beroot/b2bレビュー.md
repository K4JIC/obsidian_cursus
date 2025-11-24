
fnakamura->tnaito

Mandatory] # Project overview ✅仮想マシンの目的について、つよつよハードウェアを論理的に分割して仮想マシンとして利用できるという説明、良かったです。実際に、AWS や GCP といったクラウドコンピューティングサービスは仮想化技術に基づいているそうです。 ~~▲apptitude と apt は両方共通して package manager (パッケージマネジャー) であるという用語は覚えていて損はないと思います。 ❌AppArmor - 「セキュリティ」系のソフトウェアであることを押さえておきましょう。AppArmor についての参考資料 https://gihyo.jp/admin/serial/01/ubuntu-recipe/0798  #~~ Simple setup ▲ UFW と SSH の起動時ステータスの確認について - systemctl を使って、それらが起動していることを明快に示しましょう。 ~~❌OSの情報は、設定ファイルあるいはコマンドで確実に確認できるので、それらで明確に提示しましょう。 # User ❌“The subject requires that a user with the login of the student being evaluated exists on the virtual machine. Check that it has been added and that it belongs to the "sudo" and "user42" groups.” とある通り、レビュイーのユーザが sudo と user42 グループに含まれていないといけないですが、id -nG tnaito で該当のグループが確認できませんでした。 ✅パスワードポリシーの説明 OK です。login.defs 覚えておきましょう。 # Hostname and partitions ✅ホスト名の変更、人手で設定ファイルを変更する方法でできていて OK です。hostnamectl hostname <ホスト名> でもいけます。 # sudo ❌新しいユーザを sudo グループに追加する方法を確実に押さえておきましょう。usermod などが使えます。 #~~ UFW / Fireword ▲UFW についての説明、iptables との兼ね合いで説明すると明確になります。 ❌”Add a new rule to open port 8080. Check that this one has been added by listing the active rules. Finally, delete this new rule with the help of the student being evaluated. If something does not work as expected or is not clearly explained, the evaluation stops here.” この操作ができるようにしておきましょう。 # SSH ▲telnet などとの関係でより明確に説明できるとベターです。 # monitoring ✅/etc/systemd/system 以下の設定を触ることでスクリプト監視を実装していました。 ▲1分ごとの実行への変更に少し手間取っていたので、cron によらない方法でやるなら、ご自身で設定の仕方を確実に押さえておく必要があります。 ▲cron 以外の方法でやるにしても subject の要件にしっかりしたがっていたようなので、スクリプト名も monitorinig.sh で統一しましょう。 **********伝達事項********** - 説明系のレビュー項目について、十分な説明ができているか、確認してください。 - monitoring.sh の中身について、詳細に質問して確認してください。 - 次のレビューの 1/3 回目くらいはカンペ等使っていてもいいと思いますが、2/3 回目以降はそらで説明、作業できることを確認したいです。 ****************************** 2時間のおよぶレビュー、お疲れさまでした〜！

stanaka->tafujise

間違って起動してしまいすみませんでした。 【評価】説明できない箇所、示せない項目がありましたので、説明できないフラグを適用しました。他の人が躓くところをいくつかできていたのは素晴らしかったです。 【改善点】signature.txtにはハッシュ値のみを保存するようにしましょう。~~rootユーザーのパスワード有効期限を変更しましょう。パスワードの変更コマンドを覚えておきましょう。PAMの説明ができるようにしましょう。設定ファイルを適切に使いましょう。evaluatingグループは評価中に作成しましょう。ユーザーの切り替えなどのコマンドを勉強しましょう。LVMやパーティションについて説明できるようにしましょう。sudoの設定はvisudoを使いましょう。それぞれの項目について、説明と実際に示せるようにしましょう。~~sudoのアーカイブやsudoreplayについて調べてください。UFWの説明とreloadについて調べましょう。sshプロトコルの説明ができるようにしましょう。NAT,Bridge Adaptor, Hostonly Adaptorについて調べてみましょう。Port Forwerdingを説明できるようにしましょう。monitoring.shの複数ヶ所で不備がありました。修正してください。cronを停止させる方法を調べてください。 【総評】課題に書かれている用語、使い方などしっかり調べると理解が深まると思います。自分が何を設定しているのか一つひとつ確認しながら作業し、すべてをメモしておくと良いです。長い道のりですが頑張ってください。長時間お疲れ様でした。

stanaka->kkono

passwordの有効期限の確認コマンドchageを説明できませんでした。既存ユーザとルートユーザの変更がされていませんでした。sanapshotを使っていない証明ができるようにしましょう。apparmorとchmodの違い、MACとDACで調べて見ると良いです。PAMの説明ができる必要があります。hostnameを変更する際は、sudoが使えるようにするために、/etc/hostsを変更しておきましょう。LVMの説明がもう少しできるとなお良いです。パーティションについて理解がもう少し深まると良いですが、mandatoryでは十分でした。sudoの設定はvisudoを使ってやりましょう。sudoのアーカイブについて、sudoreplayで説明できるようにしましょう。logファイルの名前がsudo.configは不適切です。ttyモードの実践ができるようにしておきましょう。NATやポートフォーヮディングの説明をできるようにしましょう。hostname -IはIPv6も出てきてしまうので、パターンマッチが必要です。sudoのカウントは認証失敗を含めないほうが良いです。お疲れ様でした。

stanaka->tasugiya

序盤の説明は重要な点がちゃんと含まれていたので問題ありませんでした。一部知らないこともあったと思いますが、記憶と推測で近い答えが出せていたので、OKと判断しました。パーティションの構成について、サイズが異なるのは致命的であるという説明をしました。（MB、MiB）パーティションの命名規則について解説しました。SUDOについてアーカイブが認識できていませんでしたので、この項目でNoとしました。（sudoreplay)、TTYの説明と実演も必要です。SSHではNATの説明と、ポートフォーワーディングの説明ができる必要があります。monitoring.shでは、物理プロセッサの数はコア数です。hostname -I はIPv6も出てくるので、パターンマッチが必要です。sudo のカウントは認証失敗を含むべきではないと思います。全体的には良かったと思いました。お疲れ様でした。

stanaka->ykuwata

snapshotを使っていないという確認の方法は、VBoxManage snapshot "Machine Name" listです。確認できるようにしましょう。Apparmorの説明が不十分でした（MAC、DAC）。パスワードの設定に関わるPAMが説明できませんでした。sudoではTTYモードの説明と、実演ができる必要があります。sudoのアーカイブはログファイルとは別なので、認識しておく必要があります。（sudoreplay）monitoring.shについて、物理プロセッサの数はCore(s) per socketです。hostname-IはIPv6もでてきてしまう可能性がありますので、パターンマッチする必要があります。sudoのカウントについては、実行されたコマンド数であり、認証失敗は実行されていないのでカウントすべきではないと考えています。seqからカウントを取り出し、算術展開で$((36#SEQ)で十進数に変換できます。 SSHのパートでは使っているネットワークについて、説明できる必要があります。（NAT） 説明など、不足部分もありましたので、サブジェクトに書いてあることの意味、使ったサービス及びその設定について理解を深めると、誰にでもわかる説明ができると思いますので、頑張ってください。お疲れ様でした。

stanaka->tyokoyam

hash値のとり方が　sha256sumではなくsha1sumで取得するべきで、hash値も変化してしまっていました。 ユーザーのパスワード設定で使われるPAMは説明できなければいけません。既存ユーザーのパスワード期限が変更されていませんでした。パーティション構成において、ボリュームのサイズが異なっていました。（GiB,MiBとGB,MBの違い) ボリューム名が異なっていました。（var-----logとvar--log）これは設定ミスです。sudoのルール設定はvisudoで設定すべきです。ルールが重複していました。sudoのアーカイブについて知る必要があります(sudoreplayを調べてください)。TTYがなにか、TTYがなぜ必要か、TTYモードが有効であることの実演ができる必要があります。UFWのルール番号はufw status numberedで確認できます。sshとsshdの違い、NATの説明が必要です。monitoring.shについて。物理プロセッサーの参照項目が間違っていました。論理プロセッサとの関係を説明できる必要があります。LVMはアクティブかどうか確認しましょう。hostname -IはIPv6 も表示されてしまいます。sudoが試された回数ではなく、sudo付きで実行されたコマンド数を数えるべきだと私は考えています。 コマンドの入力や説明などもう少ししらべてからリトライすると、いい評価がもらえると思いますので頑張ってください。お疲れ様でした。
