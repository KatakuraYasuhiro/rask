# README

## What is rask?

raskとは，タスクを管理する機能と議事録を作成する機能を持つアプリケーションである．
raskには以下のような特徴を持つ．
- タスク管理は[Getting Things Done(GTD)](https://www.amazon.co.jp/Getting-Things-Done-Stress-Free-Productivity/dp/0143126563)に対応
- 議事録はMarkdownを用いて作成

## Setup

rask は Docker を使用して起動できるスクリプトを用意している．
このため，計算機上では Docker が使用できることを前提とする．
### Clone rask
GitHubよりraskのリポジトリをcloneし，リポジトリ内へ移動する．
```
$ git clone https://github.com/nomlab/rask.git
$ cd rask
```
### Setup config files
- 以下のコマンドを使用し，.env.example から .envをコピーする．
  ```
  $ cp .env.example .env
  ```

- OAuth App の作成
  https://github.com/settings/developers にアクセスし，OAuth App を作成する．
  この際，Homepage URL には rask のホームページのURLを入力し，Authorization callback URL には <raskのホームページのURL>/auth/github/callbackを入力する．
  例えば手元の計算機で環境を構築し，3000番ポートを使用する場合，HomePage URL には http://localhost:3000 を設定し，Authorization callback URL には http://localhost:3000/auth/github/callback を設定する．

- client id，client secret，および allowed team id の設定
  .envの内容は以下のようになっている．
  ```
  1 GITHUB_CLIENT_ID=XXXXXXXXXXXXXXXXXXXX
  2 GITHUB_CLIENT_SECRET=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
  3 OAUTH_GITHUB_ALLOWED_TEAM_ID=XXXXX
  ```

  - GITHUB_CLIENT_ID には先ほど作成した OAuth App の Client ID を入力する．
  - GITHUB_CLIENT_SECRET には OAuth App の Client secrets を入力する．
    ここで，新しい Client secrets を作成する場合，「Generate a new client secret」より作成する．
  - OAUTH_GITHUB_ALLOWED_TEAM_ID は以下のコマンドを実行し，取得したidを入力する．
    ```
    curl -H "Authorization: token <access_token>" https://api.github.com/orgs/<your_organization>/teams
    ```
    上記のコマンドのうち，\<access_token> には GitHub ユーザの Access Token を入力する．
    ここで，新しい Access Token を作成する場合，Settings から Developer settings -> Personal access tokens と移動し，New personal access token より新しい Token を作成する．
    この際，Select scopes より read:org にチェックを入れる必要がある．
    また，\<your_organization> には GitHub ユーザが所属するチーム名を入力する．

## Development
rask の開発には docker-compose を使用する．
このため，docker-compose をインストールし，以下のコマンドを用いて開発を行うとよい．

- コンテナの起動
  ```
  $ make up
  ```
- 依存関係のインストール，データベースのマイグレーション
  ```
  $ make setup
  ```

- サーバ起動
  ```
  $ make start
  ```

- テスト実行
  ```
  $ make rails-test
  ```
### Production
- コンテナイメージの作成
  ```
  $ scripts/setup-docker.sh <UserName>
  ```
  上記のコマンドのうち，\<UserName> は rask のコンテナを起動するユーザである．
​
  - Why does \<UserName> need?

    Dockerを用いてファイルを作成したとき，既定だと所有者がrootユーザで作成される．
    このため，データベースを保存するファイルがrootユーザで作成されてしまい，他のユーザがデータベースを使用できない．
    そこで，```scripts/setup-docker.sh``` と ```scripts/rask-docker.sh``` を用いることで，所有者を指定してデータベースが作成し，raskを使用できる．
    これにより，root以外のユーザでもデータベースを使用できるようになる．
​
- rask の起動
  ```
  $ scripts/rask-docker.sh start [-p PORT]
  ```
  上記のコマンドのうち，```PORT``` は rask のコンテナをバインドするポート番号である．
  デフォルトでは 3000 番で起動する．
  コンテナが起動した場合，コンテナ名は ```rask-<PORT>```(デフォルトは```rask-3000```)となる．
​
- rask の停止
  ```
  $ scripts/rask-docker.sh stop [ContainerName]
  ```
  上記のコマンドのうち，```ContainerName``` は停止させる rask のコンテナ名である．
  ```ContainerName``` が指定されていない場合，デフォルトでは```rask-3000``` が停止する．
  ただし，```rask-3000``` 以外のコンテナが起動していた場合は停止しない．

- 起動中の rask のコンテナの確認
  ```
  $ scripts/rask-docker.sh status
  Running container(s):
  rask-3030
  rask-3000
  ```

- コンテナの再起動
  ```
  $ scripts/rask-docker.sh restart [ContainerName]
  ```
  上記のコマンドのうち，```ContainerName``` は再起動させる rask のコンテナ名である．
  ```ContainerName``` が指定されていない場合，デフォルトでは```rask-3000``` が再起動する．
  ただし，```rask-3000``` 以外のコンテナが起動していた場合は再起動しない．