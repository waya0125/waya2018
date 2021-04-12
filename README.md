# Profile-Readme-WakaTime
<p align=center>
    <img src="https://img.shields.io/twitter/url?label=Twitter&style=social&url=https%3A%2F%2Ftwitter.com%2Fwayamoti2015" alt="Twitter">
    <img src="https://github.com/avinal/avinal/workflows/Build%20Graph/badge.svg" alt="Build">
    <img src="https://wakatime.com/badge/github/waya2018/waya2018.svg" alt="Time Tracked">
</p>

WakaTimeを使ってコーディング活動を記録している場合。これを棒グラフとしてREADMEに追加したり、ブログやポートフォリオに埋め込むことができます。このアクションを任意のリポジトリに追加するだけで、完成です。私の場合は以下の通りです。

## My WakaTime Coding Activity
<img src="https://github.com/avinal/avinal/blob/main/images/stat.svg" alt="Avinal WakaTime Activity"/>

## How to add one to your README.md
1. まず、WakaTime API Keyを取得します。あなたの[WakaTime](https://wakatime.com)のアカウント設定から取得できます。
2. 2. WakaTime API KeyをRepository Secretに保存します。Settings]タブをクリックしてそれを見つけます。シークレットの名前は **WAKATIME_API_KEY** のままにしておきます。
3. 3. レポのREADME.mdに以下の行を追加します。
```html
    <img src="https://github.com/<username>/<repository-name>/blob/<branch-name>/images/stat.svg" alt="Alternative Text"/>
    Example: <img src="https://github.com/avinal/avinal/blob/main/images/stat.svg" alt="Avinal WakaTime Activity"/>
    ```
    この方法を使えば、Webページにも埋め込むことができます。*マークダウン方式の画像挿入は使用しないでください。動作しないことがあります。*

4. **Action**タブをクリックし、**Set up a workflow yourself**を選択します。
5. 開いたファイルに以下のコードをコピーします。**WakaTime Stat**をマーケットプレイスタブで検索すると助けになります。
```yml
name: WakaTime status update 

on:
  schedule:
    # Runs at 12 am  '0 0 * * *'  UTC
    - cron: '1 0 * * *'

jobs:
    update-readme:
    name: Update the WakaTime Stat
    runs-on: ubuntu-latest
    steps:
    # 最新の安定版リリースには avinal/Profile-Readme-WakaTime@<latest-release-tag> を使用してください。
    # 以下の行は、masterという単語とタグ番号を除いて、変更しないでください。
    # このプロジェクトをフォークした場合は、代わりに <username>/Profile-Readme-WakaTime@master を使うことができます。
    - uses: avinal/Profile-Readme-WakaTime@master
        with:
        # WakaTimeのAPIキーはシークレットに保存されていますので、ここに直接ペーストしないでください。
        WAKATIME_API_KEY: ${{ secrets.WAKATIME_API_KEY }}
        # 自動githubトークン
        GITHUB_TOKEN: ${{ github.token }}
        # Branch - 新しいGitHubリポジトリでは、デフォルトのブランチが "main "になっているので、その場合はmainに変更します。
        BRANCH: "master"
        # マニュアル・コミット・メッセージ - ここに自分のメッセージを書く
        COMMIT_MSG: "Automated Coding Activity Update :alien:"

```
6. このワークフローを自動的に実行するには、午前12時（UTC）までお待ちください。または、Actionタブで強制実行することもできます。または、 `on:` の下に以下の行を追加して、プッシュされるたびに実行することもできます。12 AM UTCで検索すると、あなたのタイムゾーンでの同等の時間がわかります。
```yml
on:
    push:
        branches: [ master ]
    schedule:
        - cron: '1 0 * * *' 
```

## Implementation Details
このGitHub Actionは3つのパートに分かれています。本当はDockerを使いたくなかったのですが、どうやら使わないとうまくいかないようです。少し技術的な話をします。3つのパートは以下の通りです。

1. *[main.py](https://github.com/avinal/Profile-Readme-WakaTime/blob/master/main.py)* pythonスクリプトです。このスクリプトには多くの手順が含まれています。
    * [Getting JSON data file via WakaTime API](https://github.com/avinal/Profile-Readme-WakaTime/blob/master/main.py#L52) 
    ```python
        def get_stats() -> list:
        ...
        return data_list
    ```
    この関数は、受け取ったjsonファイルを解析し、有用なデータをリストとしてスクレイピングします。スクラップされるデータは、言語リスト、各言語に費やした時間、時間の割合、開始日と終了日です。このアクションでは、言語の数を5つに制限していますが、その数を増やすことは非常に簡単です。
    * [Setting the Timeline](https://github.com/avinal/Profile-Readme-WakaTime/blob/master/main.py#L13)
    ```python
        def this_week(dates: list) -> str:
        ...
        return f"Coding Activity During: {week_start.strftime('%d %B, %Y')} to {week_end.strftime('%d %B, %Y')}"
    ```
    ここでは、前回の関数で取得した開始日と終了日を使ってタイムラインを設定します。jsonの日付はUTCで提供されているため、以下のようになります。
    ```json
        date:	"YYYY-MM-DDTHH:MM:SSZ"
    ```
    単純な日付だけにしました。現在の時刻をシステムから取得して手動で設定することもできます。しかし、この方法には欠陥があります。しかし、この方法では、最新のJSONを受信し、リクエストが成功したことを確認します。異常があれば、リクエストに失敗したことになります。
    * [Creating a bar graph](https://github.com/avinal/Profile-Readme-WakaTime/blob/master/main.py#L21)
    ```python
        def make_graph(data: list):
        ...
        savefig(...)
    ```
    最後に、グラフを生成して画像として保存します。この関数では、最初のステップでスクレイピングしたデータを使用します。`matploylib` を使った棒グラフの作成は簡単です。デコレーションはちょっと難しいですね。このグラフは GitHub の外観に合わせたかったので、GitHub が言語に色をつけるように棒グラフにも色をつけることにしました。このデータは `colors.json` として保存されています。多くの言語は、WakaTimeと比べてGitHubでの綴りが微妙に異なります。そのため、いくつかの言語はデフォルトの色で表示されています。これは、その言語に気づいて手動で色を変更すれば改善されます。最後に、このグラフはSVGとPNGの両方で保存されます。SVGはズーム可能なページに置くのに適しています。
  
2. *[entrypoint.py](https://github.com/avinal/Profile-Readme-WakaTime/blob/master/entrypoint.sh)* シェルスクリプトです。このシェルスクリプトは、repoをクローンし、イメージをコピーし、変更をマスターにプッシュします。いくつかの問題がありました。まず、認証です。これはGitHub Tokenを使ってリモートリポジトリのアドレスを使うことで解決しました。また、GitHubではユーザー名とメールアドレスがないとコミットできないようです。そこで、**github-actions** というボットメールを使いました。
```bash
    remote_repo="https://${GITHUB_ACTOR}:${INPUT_GITHUB_TOKEN}@github.com/${GITHUB_REPOSITORY}.git"
    git config user.email "41898282+github-actions[bot]@users.noreply.github.com"
    git config user.name "GitHub Actions"
```
*41898282* は、github-actions のボットに割り当てられた ID です。どこで見つけたかは聞かないでください🙂。

もうひとつの問題は、`${GITHUB_REPOSITORY}` で提供される *username/repository-name* の組み合わせからリポジトリ名を切り離すことでした。GitHub は、リポジトリ名だけを直接取得する方法を提供していません。そこで、*Internal Field Seperator* を使用しました。これは配列を返すもので、PythonやJavaの`split()`コマンドに似た動作をします。
```bash
    # '/' is the seperator
    IFS='/' read -ra reponame <<< "${GITHUB_REPOSITORY}"
    # returned {username, repository}
    repository="${reponame[1]}"
```
この後のコマンドはすべて単純です。追加したファイルをコミットし、プッシュします。

3. *[Dockerfile](https://github.com/avinal/Profile-Readme-WakaTime/blob/master/Dockerfile)* **重要** この状態になるまでには、かなりの時間がかかりました（笑）。ここからが魔法の始まりです。コンテナの中で`ubuntu:latest`を動かしています。まずディストリビューションをアップデートします。次に、必要なpythonパッケージをインストールします。最後にpyhtonスクリプトとシェルスクリプトを起動します。

ほとんど不可能な問題がありました。 *Dockerコンテナ内で生成されたファイルにアクセスするにはどうすればよいか* という記事を何百も探しましたが、見つかりませんでした。しかし、ついに私は回避策を見つけました（そうでなければ、今頃あなたはこれを読んでいないでしょう🤣）実際、各コマンドは別々の仮想サブコンテナで実行されます。コマンドが終了すると、その出力も失われますが、複数のコマンドをまとめて実行した場合には失われません。少なくとも、すべてのコマンドが終了するまでは。生成されたファイルはクラブ化された次のプロセスで利用できます。私はpythonスクリプトの実行とシェルスクリプトの実行を組み合わせることでこれを行いました。
```dockerfile
    CMD python3 /main.py && /entrypoint.sh
```
この部分は最も小さいものですが、このアクションを開発する際に最も多くの時間と試みを費やしました。

最終的にプロジェクトは完成しましたが、誰かがより良いバージョンを開発したり、私の経験から助けを得たりできるように、挑戦したことをすべて書き残しておきたかったのです。お楽しみいただけましたら幸いです。