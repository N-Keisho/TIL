# GitHub ActionにおけるYAML
GitHub ActionのWorkflowの設定ファイルはYAMLで書く．
YTMLファイルにワークフローとして，トリガー，ジョブ，ステップを書いていく．
Diary用に作ったものを部分部分を抜粋してメモする．

# トリガー
どのタイミングで実行するかを設定できる．
```YAML
on:
  schedule:
    - cron: '21 0  * * *'
```
トリガーは一つではなく，複数同時に設定することも可能．
詳細は以下．

https://docs.github.com/ja/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows#schedule

# ジョブ
具体的な操作を書くところ
```YAML
jobs:
  make_diary:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Test
        run: echo Hello World!
```
## 実行環境
ubuntuの最新版で動かすときはこれをかく
```YAML
runs-on: ubuntu-latest
```
## ステップ
操作を段階に分けて記述する．一つのステップに名前を付けられ，GitHubActionの表示も分かれる．
細かく分けることでどこでエラーがおきているかがわかりやすくなる．
### リポジトリを持ってくるとき
```
steps:
      - name: Checkout repository
        uses: actions/checkout@v4
```
GitHub Acitonの実行環境にはデフォルトでリポジトリが入っていない．
そのためクローンする必要があるが，これを書くだけで簡単に持ってこれるようになっている．
### UNIX系のコマンドを実行するとき
```YAML
      - name: Test
        run: echo Hello World!
```
run:の後に書くと実行される．
### 変数を使いたい時
変数の保存と使用は少し癖がある．
保存したステップの中ですぐ使おうとするとエラーが出た．
ステップ内は並列的に処理されているかもしれないので，変数を使うときはステップを分けるといいかも．
```YAML
- name: Save
  run: echo "YEAR=2024" >> $GITHUB_ENV
- name: Use
  run: echo ${{env.YEAR}}
```
### 日付を取得するとき
これはUNIXコマンドなのでYAMLは関係ない
```YAML
- name: Get Date
        env:
          TZ: 'Asia/Tokyo' # タイムゾーン指定
        run: |
          echo "YEAR=$(date +'%Y')" >> $GITHUB_ENV
          echo "MONTH=$(date +'%m')" >> $GITHUB_ENV
          echo "DATE=$(date +'%d')" >> $GITHUB_ENV
```
### GitHubにプッシュするとき
ユーザ名とメールアドレスを設定しないといけない．
またリポジトリの設定も変更しないと，GitHubActionからのプッシュができないので注意．
Setting → Actions → General → Workflow permissions から Actionsの権限を Read and Write に変更する．
secrets.はリポジトリで設定したシークレットの環境変数
```YAML
- name: Set Git Uer Email
  run: git config user.email ${{ secrets.EMAIL }}

- name: Set Git Uer Name
  run: git config user.name ${{ secrets.USERNAME }}

- name: Git Add
  run: git add .

- name: Git Commit
  run: git commit -m "generated"

- name: Git Push
  run: git push
```
# 全体
```YAML
name: Make Diary Template
on: 
  schedule:
    - cron: '21 0  * * *'
jobs:
  make_diary:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Get Date
        env:
          TZ: 'Asia/Tokyo' # タイムゾーン指定
        run: |
          echo "YEAR=$(date +'%Y')" >> $GITHUB_ENV
          echo "MONTH=$(date +'%m')" >> $GITHUB_ENV
          echo "DATE=$(date +'%d')" >> $GITHUB_ENV

      - name: Set FILENAME
        run: |
          echo "FILENAME=${{ env.YEAR }}${{ env.MONTH }}${{ env.DATE }}" >> $GITHUB_ENV
          echo ${{ env.FILENAME }}

      - name: Make Folder
        run: mkdir -p ${{ env.YEAR }}/${{ env.MONTH }}

      - name: Make File 
        run: touch ${{ env.YEAR }}/${{ env.MONTH }}/${{ env.FILENAME }}.md

      - name: Write Template
        run: |
          echo "# ${{env.FILENAME}}" >> tmp
          echo "" >> tmp
          echo "## よかったこと" >> tmp
          echo "" >> tmp
          echo "## わるかったこと" >> tmp
          echo "" >> tmp
          echo "## 明日への意気込み" >> tmp
          echo "$(cat tmp)" >> ${{ env.YEAR }}/${{ env.MONTH }}/${{ env.FILENAME }}.md
          rm tmp

      - name: Show File
        run: cat ${{ env.YEAR }}/${{ env.MONTH }}/${{ env.FILENAME }}.md

      - name: Set Git Uer Email
        run: git config user.email ${{ secrets.EMAIL }}

      - name: Set Git Uer Name
        run: git config user.name ${{ secrets.USERNAME }}

      - name: Git Add
        run: git add .

      - name: Git Commit
        run: git commit -m "generated"

      - name: Git Push
        run: git push
```
# 参考記事
これがわかりやすかった↓

https://zenn.dev/praha/articles/9e561bdaac1d23　

https://qiita.com/shun198/items/14cdba2d8e58ab96cf95

https://zenn.dev/shibayan/articles/744c3c7c3faa4f

https://ayousanz.hatenadiary.jp/entry/GitHub_Actions%E3%81%8B%E3%82%89push%E3%81%99%E3%82%8B%E9%9A%9B%E3%81%AB%E6%A8%A9%E9%99%90%E3%81%A7%E3%82%A8%E3%83%A9%E3%83%BC%E3%81%8C%E5%87%BA%E3%81%9F%E5%A0%B4%E5%90%88%E3%81%AB%E7%A2%BA%E8%AA%8D%E3%81%99
