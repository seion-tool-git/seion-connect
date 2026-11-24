# 整音キットをつなげる（Claude 向けの手順）

この文書は、メンバーが Claude に「整音キットをつなげて」と頼んだときに **Claude が読んで実行する手順**です。
相手は音や PC の専門家ではありません。次を守ってください。

- 操作はすべて Claude が行う。相手にターミナルやコマンドを打たせない。
- 相手に聞くときは **はい／いいえで答えられる聞き方**にする（例:「ブラウザで GitHub のページを開きます。よいですか？」）。
- 相手にしてもらうのは、ブラウザでのボタン操作とコードの入力だけ。パスワード・確認コード・鍵を Claude が受け取ったり入力したりしない。
- 1 つ終わるごとに短く「できました」と伝え、次へ進む。途中で失敗したら、原因を一言で伝えて、もう一度試すか聞く。
- すでに済んでいる手順は確認だけして飛ばす（やり直しのときも同じ手順で OK）。

組織名（ORG）: `seion-tool-git`　キット: `seion-tool-git/seion-kit`　学びノート: `seion-tool-git/seion-learn`

## 1. Mac の準備
1. `xcode-select -p` で開発ツールがあるか確認する。無ければ `xcode-select --install` を実行し、
   「Mac に『インストール』の画面が出ます。『インストール』を押して、終わったら教えてください。終わりましたか？」と聞く。
2. Python 3 が動くか確認する（`python3 --version`）。

## 2. GitHub を使えるようにする
1. GitHub CLI（gh）が無ければ、GitHub 公式のリリースから入れる（管理者権限は不要）:
   - `https://api.github.com/repos/cli/cli/releases/latest` から macOS 用の zip（名前に `macOS_universal` か `macOS_arm64` を含むもの）を取得し、
     `~/.local/gh/` に展開して `~/.local/bin/gh` にリンクする。以降 gh はこのパスで呼ぶ。
**ブラウザ**: アカウント作成とログインの許可は、**同じブラウザ**で行う（別のブラウザだとログインし直しになる）。
Claude アプリの内蔵ブラウザ（Browser pane のツール）が使えればそれを使い、使えなければ普段のブラウザを使う。

2. GitHub のアカウントがあるか聞く:「GitHub のアカウントはありますか？」
   - いいえ → ブラウザで `https://github.com/signup` を開き、
     「GitHub の登録画面を開きました。**招待メールが届いたのと同じメールアドレス**で登録してください（Google アカウントで登録しても OK）。
     メールアドレス・パスワード・確認コード・パズルはご自身で入力してください。登録できましたか？」と聞く。
     Claude は入力欄に何も入れない。プランを聞かれたら Free、アンケートは飛ばしてよいと伝える。
   - 2 段階認証の設定を求められたら、画面に沿って設定してもらう（スマホのアプリなど）。
3. ログイン: `GH_BROWSER=true ~/.local/bin/gh auth login --web --hostname github.com --git-protocol https < /dev/null` をバックグラウンドで実行し、出力の 8 桁のコード（`XXXX-XXXX`）を読む。
   - 手順 2 と同じブラウザで `https://github.com/login/device` を開き、コードを相手に大きく伝えて
     「このコードを入れて Continue、次の画面で緑の『Authorize github』を押してください。押せましたか？」と聞く。
     コードの入力と Authorize は相手が行う。
   - 終わったら `~/.local/bin/gh auth setup-git` を実行する。
4. 組織への招待を受ける: `~/.local/bin/gh api -X PATCH user/memberships/orgs/seion-tool-git -f state=active`
   - 失敗（招待が見つからない）したら、まず「招待メールと違うメールアドレスで登録した可能性があります。招待メールの『Join』ボタンを、いまのブラウザで押してもらえますか？」と聞いてもう一度試す。
   - それでも失敗したら「招待がまだ届いていないようです。管理している人に、GitHub への招待をお願いしてください」と伝えて止める。

## 3. キットを取り込む
1. `git clone https://github.com/seion-tool-git/seion-kit.git ~/seion-kit`（既にあれば `git -C ~/seion-kit pull`）
2. `git clone https://github.com/seion-tool-git/seion-learn.git ~/seion-learn`（既にあれば `git -C ~/seion-learn pull`）
3. 学びノートに書き込むときの名前を設定する（GitHub の匿名アドレスを使う）:
   `login=$(~/.local/bin/gh api user -q .login)`、`id=$(~/.local/bin/gh api user -q .id)` として
   `git -C ~/seion-learn config user.name "$login"`、`git -C ~/seion-learn config user.email "$id+$login@users.noreply.github.com"`
4. `zsh ~/seion-kit/tools/setup.sh`（「seion ok」と出れば OK。数分かかることがある）
5. スキルを登録する: `mkdir -p ~/.claude/skills && ln -sfn ~/seion-kit/skills/premiere-seion ~/.claude/skills/premiere-seion`
6. `zsh ~/seion-kit/tools/update.sh` で最後の確認（「ready」と出れば OK）

## 4. Premiere とのつながりを確認
1. Premiere の MCP ツール（`verify_premiere_connection`）が使えるか確認する。
   - 使えれば「Premiere ともつながっています」と伝える。
   - 使えなければ「Premiere 側の準備（MCP Bridge）がまだのようです。Premiere を開いて『ウィンドウ > エクステンション > MCP Bridge』を表示してから、もう一度『Premiere の接続を確認して』と言ってください」と伝える。

## 5. 終わり
「整音キットをつなげました。いったんこの会話を閉じて、新しい会話を開いてから『このシーケンスを整音して』と頼んでください。」と伝える。
