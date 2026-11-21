# つまずきログ（React_laravel_PF）

開発中につまずいたことを、**症状 → 原因 → 解決 → 学び** の順に記録する。
「何が分からなかったか」を言葉にしておくと、同じミスを防げて、面接で説明する練習にもなる。

- 新しいものを上に追加する
- エラーメッセージは省略せずにそのまま貼る
- 「自分の言葉で」の欄は、AI を見ずに自分で書く

## 目次

| No | 日付 | 分類 | タイトル |
|---|---|---|---|
| 006 | 2026-11-21 | Git | push で認証エラー（HTTPS ではパスワードが使えない） |
| 005 | 2026-11-21 | Git | `git status` に `.idea/` が出た |
| 004 | 2026-11-21 | ターミナル | `cd backend` で no such file or directory |
| 003 | 2026-11-21 | Laravel | `php artisan migrate` で Could not open input file: artisan |
| 002 | 2026-11-21 | Laravel | 1回目の `php artisan migrate` でエラー（原因未確認） |
| 001 | 2026-11-21 | 環境 | localhost:8000 で pf-shop が表示された |

---

## 記入テンプレート

```markdown
## No.000 タイトル（日付）

**分類**：環境 / Laravel / React / Git / DB / ターミナル / その他

### 症状
何をしようとして、何が起きたか。

（エラーメッセージ）

### 原因

### 解決
（実行したコマンドと、その意味）

### 学び・再発防止

### 自分の言葉で
（AI を見ずに、人に説明するつもりで書く）
```

---

## No.006 push で認証エラー（2026-11-21）

**分類**：Git

### 症状
GitHub に最初の push をしたら、ユーザー名とパスワードを聞かれ、入力しても失敗した。

```
remote: Invalid username or token. Password authentication is not supported for Git operations.
fatal: Authentication failed for 'https://github.com/Ryo-w1004/React_laravel_PF.git/'
```

### 原因
送り先を HTTPS 形式の URL（`https://github.com/...`）で登録していた。
GitHub は、push のときに **GitHub のログインパスワードを受け付けない**。
HTTPS で push するにはトークンが必要。

一方、pf-shop は SSH 形式の URL（`git@github.com:...`）で登録していた。
SSH は Mac に入っている鍵ファイルで本人確認するので、パスワードの入力が不要。

### 解決
```bash
cd ~/dev/pf-shop && git remote -v
```
→ pf-shop の送り先を表示。`git@github.com:` で始まっていたので、SSH を使っていると分かった。

```bash
cd ~/dev/React_laravel_PF && git remote set-url origin git@github.com:Ryo-w1004/React_laravel_PF.git
```
→ 登録済みの `origin` の URL を SSH 形式に書き換える。

```bash
git push -u origin main
```
→ 成功。

### 学び・再発防止
- 新しいリポジトリを作ったら、GitHub の画面で **SSH タブの URL** をコピーする
- SSH 鍵は「Mac と GitHub アカウント」の間で1回登録すれば、どのリポジトリでも使える
  - `~/.ssh/id_ed25519`（秘密鍵：Mac から出さない）
  - `~/.ssh/id_ed25519.pub`（公開鍵：GitHub の Settings → SSH and GPG keys に登録）
- 確認コマンド：`ssh -T git@github.com`（`Hi Ryo-w1004!` と出れば OK）
- `origin` はリポジトリごとの `.git/config` に登録される名前なので、pf-shop と同じ名前でもぶつからない

### 自分の言葉で


---

## No.005 `git status` に `.idea/` が出た（2026-11-21）

**分類**：Git

### 症状
`git add .` のあと `git status` を見たら、`.idea/` の中のファイルがコミット対象に入っていた。
`.env` も除外されていないのではないかと不安になった。

### 原因
- `.env`、`vendor`、`node_modules` は一覧になく、**正しく除外されていた**（雛形の `backend/.gitignore`・`frontend/.gitignore` で除外済み）
- `.env.example` は値が空のひな形なので、公開してよいファイル
- `.idea/` は PhpStorm が自動で作る**自分の PC 用の設定**。直下に `.gitignore` がなかったので除外されていなかった

### 解決
```bash
printf ".idea/\n.DS_Store\n" > .gitignore
```
→ 直下に `.gitignore` を作り、`.idea/` と `.DS_Store` を書き込む（`\n` は改行、`>` はファイルに上書き）。

```bash
git rm -r --cached .idea
```
→ `.idea` を Git の管理からだけ外す。`--cached` を付けると PC のファイルは消えない。`-r` はフォルダごと。

```bash
git add . && git status
```
→ `.idea/` が消え、`.gitignore` が追加されたことを確認。

### 学び・再発防止
- `git add .` のあとは、コミット前に必ず `git status` で中身を確認する
- 確認するポイント：`.env`（パスワード）、`vendor`・`node_modules`（ライブラリ本体）、エディタの設定フォルダ
- 追記するときは `>>`、上書きするときは `>`（間違えると既存の行が消える）

### 自分の言葉で


---

## No.004 `cd backend` で no such file or directory（2026-11-21）

**分類**：ターミナル

### 症状
```
user@web2 backend % cd backend && php artisan migrate
cd: no such file or directory: backend
```

### 原因
すでに `backend` の中にいたため、その中にさらに `backend` フォルダを探して見つからなかった。

### 解決
`cd` をやめて、`php artisan migrate` だけを実行した。

### 学び・再発防止
- プロンプトの `user@web2 ○○ %` の ○○ が、今いるフォルダ名
- コマンドを打つ前に、プロンプトで今いる場所を確認する

### 自分の言葉で


---

## No.003 Could not open input file: artisan（2026-11-21）

**分類**：Laravel

### 症状
```
user@web2 React_laravel_PF % php artisan migrate
Could not open input file: artisan
```

### 原因
`artisan` は `backend` フォルダの中にあるのに、`React_laravel_PF` 直下で実行していた。
（`use84` で PHP を切り替えても変わらなかったのは、PHP のバージョンの問題ではなかったため）

### 解決
```bash
cd backend && php artisan migrate
```
→ `backend` に移動してから実行した。

### 学び・再発防止
コマンドごとに実行する場所が決まっている。

| コマンド | 実行する場所 |
|---|---|
| `php artisan ○○` | `backend` |
| `npm ○○` | `frontend` |
| `git ○○` | `React_laravel_PF` 直下 |

`Could not open input file: ○○` は「今いるフォルダに ○○ というファイルがない」という意味。

### 自分の言葉で


---

## No.002 1回目の `php artisan migrate` でエラー（2026-11-21）

**分類**：Laravel / DB

### 症状
1回目の `php artisan migrate` で、`create_users_table.php` の14行目（`Schema::create`）でエラーになった。
2回目は全部 DONE で成功した。

```
at vendor/laravel/framework/src/Illuminate/Database/Connection.php:864
...
10  database/migrations/0001_01_01_000000_create_users_table.php:14
```

### 原因
**未確認**。エラーメッセージの先頭（一番大事な1行目）を残していなかった。

### 解決
もう一度実行したら成功した。

```bash
php artisan db:show
```
→ 今つながっている DB の情報を表示。`mysql`、ポート `8889`、DB 名 `react_laravel_pf` と出れば MAMP の MySQL につながっている。
（`.env` が `sqlite` のままでも migrate は成功するので、つながり先を確認する）

### 学び・再発防止
- エラーは**先頭の1行目**に原因が書いてあることが多い。必ず先頭からコピーして残す
- 下の方の `at vendor/...` は「どこで起きたか」の情報で、原因そのものではない
- 「もう一度やったら直った」で終わらせず、何が変わったのかを確認する

### 自分の言葉で


---

## No.001 localhost:8000 で pf-shop が表示された（2026-11-21）

**分類**：環境

### 症状
React_laravel_PF の `backend` で `php artisan serve` を実行し、http://localhost:8000 を開いたら、pf-shop の画面が表示された。

### 原因
pf-shop の `php artisan serve` が別のタブで動いたままで、8000 番ポートを使っていた。
`php artisan serve` は 8000 番が使用中だと自動で **8001 番** で起動するので、React_laravel_PF は 8001 番で動いていた。

### 解決
http://localhost:8001 を開いたら、React_laravel_PF の Laravel が表示された。

ポートを使っているプロセスを調べて止める方法：
```bash
lsof -i :8000
```
→ 8000 番ポートを使っているプロセスを表示。`PID` 列がプロセス番号。

```bash
kill PID番号
```
→ そのプロセスを終了させる。

ポートを指定して起動する方法：
```bash
php artisan serve --port=8080
```

### 学び・再発防止
- `php artisan serve` を実行したら、出力の `Server running on [http://127.0.0.1:○○○○]` でポート番号を確認する
- 別のプロジェクトに移るときは、前のプロジェクトのサーバーを **Ctrl + C** で止める
- React から API を呼ぶときの URL にポート番号を書くので、ポートはそろえておく

### 自分の言葉で

