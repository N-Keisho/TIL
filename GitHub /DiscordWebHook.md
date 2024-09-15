# DiscrodでGitHubのpush時に通知を受け取る方法
DiscordのWebHookを使えば簡単にできちゃう
## 手順
1. Discordで ```サーバ設定 > 連携サービス > ウェブフック > 新しいウェブフック``` からウェブフックを作成
2. GitHubで ```Settings > WebHooks > Add webhoolk``` のページを開く
3. PayloadURLにDiscordで作成したウェブフックの ```ウェブフックURLをコピー``` でコピーされたURLを入れる
4. URLの最後に ```/github``` をつける（忘れがち）
5. GitHubの設定で ```Content typ```e を ```application/json``` に変更する
6. 必要に応じて双方の設定を変える
7. ```Add webhook``` でウェブフックを追加する
