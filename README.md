# Snyk Container ワークショップ

Snyk Container を使うとコンテナイメージ内の脆弱性を検出、修正できます。古くなってしまったベースイメージをセキュアなバージョンへ更新したり、再構築することを通じて、大量の脆弱性を開発者が一気に修正できるというのがこの製品の大きな特徴です。つまり、Snyk Container でセキュリティ対策をスケールできるのです。

コンテナ内で実行されるアプリケーションのソースコードにアクセスできるとは限りませんが、そこで使われている依存ライブラリに含まれる脆弱性は無視できません。Snyk はコンテナスキャン時に主要言語のオープンソース脆弱性を検出し、モニタリングを行います。

このハンズオンワークショップでは以下のステップをカバーします。

* [Step 1 Nodejs-Goof アプリケーションのフォーク](#step-1-nodejs-goof-アプリケーションのフォーク)
* [Step 2 GitHub インテグレーションの設定](#step-2-github-インテグレーションの設定)
* [Step 3 Docker Hub インテグレーションの設定](#step-3-docker-hub-インテグレーションの設定) 
* [Step 4 プロジェクトの追加 (コンテナイメージのスキャン)](#step-4-プロジェクトの追加-コンテナイメージのスキャン)
* [Step 5 プロジェクトの追加 (Dockerfile のスキャン)](#step-5-プロジェクトの追加-dockerfile-のスキャン)
* [Step 6 Pull Request によるベースイメージの更新](#step-6-pull-request-によるベースイメージの更新)
* [Step 7 Snyk CLI を用いたコンテナイメージのスキャン](#step-7-snyk-cli-を用いたコンテナイメージのスキャン)

## 事前準備 (以下をご用意ください)

* GitHub アカウント (パブリックであること) - http://github.com
* git CLI - https://git-scm.com/downloads
* snyk CLI - https://docs.snyk.io/snyk-cli/install-the-snyk-cli ([Snyk CLI のインストールとアップデート](https://qiita.com/ToshiAizawa/items/c090cbd525e45cc5ae51))
* Snyk アカウント - http://app.snyk.io
* Docker Hub アカウント - https://hub.docker.com/
* ローカル環境への Docker Desktop のインストールと実行 - https://www.docker.com/products/docker-desktop

__注意: ワークショップ開始前に上記の事前準備が完了していることを確認してください__

# Workshop Steps

__注: 以下のステップでは主に Mac の利用を想定していますが、Windows または Linux 上でもスクリプトを一部変更することで対応できます。__

## Step 1 Nodejs-Goof アプリケーションのフォーク

次の GitHub リポジトリにアクセスしてください - https://github.com/snyk-labs/nodejs-goof

* "**Fork**" ボタンを選択します
* フォーク先がご自身のパブリックな GitHub アカウントであることを確かめます
* "**Create fork**" ボタンを選択します

<img width="1327" alt="image" src="https://user-images.githubusercontent.com/95601557/194745394-c2d4543c-f0c5-48b1-9ab4-18c8e2e4cea7.png">

## Step 2 GitHub インテグレーションの設定

注: GitHub インテグレーションを設定済みの場合、このステップは省略できます。次のステップへ進んでください。

Snyk を GitHub に接続してリポジトリをインポートできるようにします。以下のステップを実行してください。

* http://app.snyk.io へログインする (サインアップをしていない場合はここでサインアップ)
* Integrations タブ -> Source Control -> GitHub を選択する
* クレデンシャルを設定して GitHub アカウントへ接続する

![alt tag](https://i.ibb.co/bPqqybM/snyk-starter-open-source-1.png)

## Step 3 Docker Hub インテグレーションの設定

Docker Hub と Snyk の間でインテグレーションを有効化することにより、コンテナ脆弱性の管理を開始できます。次のとおり Docker Hub への接続を行います。

* Integrations タブ -> Container Registries -> Docker Hub を選択する
* Docker Hub の Username と Access Token を入力した後、Save を選択する

Snyk が入力されたクレデンシャルを確認した後、ページはリロードされ、Docker Hub のインテグレーションに関する情報と Add your Docker Hub images to Snyk ボタンが表示されます。Docker Hub へ接続が行われたことを示すメッセージも緑色のボックス内に表示されます。また、Docker Hub への接続が失敗した場合は、失敗を伝えるメッセージが表示されます。

注: Access Token フィールドには、Docker Hub のパスワードもしくは Docker Hub で作成した [access token](https://docs.docker.com/docker-hub/access-tokens/) を入力することができます。Docker Hub アカウントに対して 2FA が有効になっている場合は、access token のみが利用できます。

![alt tag](https://i.ibb.co/hYyb7RD/snyk-container-1.png)

![alt tag](https://i.ibb.co/pWkKGmh/snyk-container-2.png)

* Snyk Container のコンテナレジストリ インテグレーションではコンテナイメージ内のアプリケーション脆弱性の検出が可能です。この機能を有効化するには次のドキュメントの指示に従ってください。

[コンテナイメージ内のアプリケーション脆弱性を検出する (Detecting application vulnerabilities in container images)](https://docs.snyk.io/products/snyk-container/getting-around-the-snyk-container-ui/detecting-application-vulnerabilities-in-container-images)

## Step 4 プロジェクトの追加 (コンテナイメージのスキャン)

Docker Hub レジストリにはすでにイメージが存在するかもしれませんが、あなたの Docker Hub アカウントに新しいイメージを追加してみましょう。

* 次のように Docker Hub にログインします。Step 3 で使ったのと同じクレデンシャルを用いてください。

```bash
$ docker login -u DOCKER_HUB_USERNAME -p YOUR_ACCESS_TOKEN_OR_PASSWORD
Login Succeeded
```

* 次のようにイメージを pull (取得) します。

```bash
$ docker pull pasapples/docker-goof
Using default tag: latest
latest: Pulling from pasapples/docker-goof
Digest: sha256:be2d6b0f5041315f632f44e3528ea513c452cc95ed4a40bc3d70f943ab293e5f
Status: Downloaded newer image for pasapples/docker-goof:latest
docker.io/pasapples/docker-goof:latest
```

* 次のコマンドを実行してイメージへ tag (タグ付け) します。

注: 以下では、DOCKER_HUB_USERNAME はあなたの Docker Hub ユーザー名に置き換えてください。

```bash
$ docker tag pasapples/docker-goof:latest DOCKER_HUB_USERNAME/docker-goof:latest
```

* 次のコマンドを実行してイメージを Docker Hub アカウントに push (送信) します。

```bash
$ docker push DOCKER_HUB_USERNAME/docker-goof:latest
The push refers to repository [docker.io/pasapples/docker-goof]
1bc5d83ccce7: Layer already exists
35bda1fbb3d0: Layer already exists
5f70bf18a086: Layer already exists
fbd39f37d7c2: Layer already exists
938fc2ad056c: Layer already exists
c24944d2eccc: Layer already exists
02a318dedbea: Layer already exists
7972420bc26e: Layer already exists
17c76043bf23: Layer already exists
496d6557f1e3: Layer already exists
867786449541: Layer already exists
92d17ee6d9da: Layer already exists
e54368741774: Layer already exists
5a6c4d956b5d: Layer already exists
86ab2c6c5d58: Layer already exists
latest: digest: sha256:be2d6b0f5041315f632f44e3528ea513c452cc95ed4a40bc3d70f943ab293e5f size: 3466
```

* Snyk Web UI へ戻り、"**Add your Docker Hub Images to Snyk**" ボタンを選択します。

* "**docker-goof**" で検索し、イメージを選択してから、"**Add Selected Repositories**"　ボタンを選択します。

![alt tag](https://i.ibb.co/mq421V8/snyk-container-3.png)

* この操作は完了までに数分を要します。待っている間、次のリンクを用いて Import の状況を確認することができます。

![alt tag](https://i.ibb.co/PQy4pzq/snyk-container-4.png)

* 完了すると、コンテナスキャンの結果が次のとおり表示されます。

![alt tag](https://i.ibb.co/NTX9KV2/snyk-container-5.png)

Snyk Container が提供されているインテグレーションを使ってイメージをスキャンすると、まずイメージに含まれるソフトウェアを検出します。これらのソフトウェアには次のものが含まれます。

1. dpkg、rpm、apk のオペレーションシステム・パッケージ
1. アンマネージドな (すなわちパッケージマネージャーを使わずにインストールされた) ソフトウェア
1. マニフェストファイルの存在により検出されたアプリケーションパッケージ

__注: Snyk は必要な情報をファイルシステムより取得するため、コンテナを実行する必要はありません。よって、スキャンを完了するためにコンテナや外部コードの実行は必要ありません。__

インストール済みソフトウェアのリストを検出すると、次に Snyk はそれらを脆弱性データベースと照合します。このデータベースは公開情報と独自の研究成果を組み合わせたものです。

* "**latest**" のリンクを選択して、スキャン結果の一部であるコンテナ脆弱性を確認しましょう

![alt tag](https://i.ibb.co/HGS4MSP/snyk-container-7.png)

ここで、ベースイメージの更新が推奨されていることに気づくはずです。ベースイメージを置き換えるだけで数多くの脆弱性を修正することができるため、これは非常に便利です。マイナーバージョンアップやメジャーバージョンアップを用いた更新の選択肢が提示されると共に、ベースイメージを更新してコンテナを再構築した場合に残される脆弱性についても情報が提供されます。

サポート対象のベースイメージについてはこの[リンク](https://snyk.io/docker-images/)でご確認ください。

検出された脆弱性は [Priority Score](https://snyk.io/blog/snyk-priority-score/) でソートされ、脆弱性ごとに以下の情報が提供されます。

1. 脆弱性を混入させたモジュール (OS、ベースイメージまたはユーザーレイヤーのいずれであるか) と、それが推移的依存パッケージの場合は直接的依存パッケージが明示される
1. パスと修正方法、脆弱な関数
1. 概要
1. エクスプロイトマチュリティ (攻撃可能性)
1. CWE と CVE へのリンクと CVSS スコア
1. ソーシャルネットワークにおけるトレンド
1. その他

## Step 5 プロジェクトの追加 (Dockerfile のスキャン)

Snyk は Git リポジトリのインポート時に、Dockerfile のスキャンを通じて脆弱性のあるベースイメージを検出することができます。これによりイメージのビルド前にセキュリティ上の問題をチェックすることができるため、脆弱性がレジストリや本番環境にたどり着く前にその問題を解決することが可能です。

Snyk は GitHub レポジトリと接続済みのため、リポジトリをインポートして Dockerfile をスキャンしてみましょう。

* Projects タブを選択する
* "**Add Project**" ボタンを選択し、続いて "**GitHub**" を選択する
* Step 1 でフォークしたリポジトリ "goof" を選択する

![alt tag](https://i.ibb.co/q9Rsxsh/snyk-starter-open-source-3.png)

__注: インポートには 1 分程度を要します。次のように、インポート中にインポートログを見ることができます__

![alt tag](https://i.ibb.co/RQsX6jZ/snyk-starter-open-source-14.png)

* インポートが完了すると、次のように Dockerfile への参照が確認できます。

![alt tag](https://i.ibb.co/1rNMFhC/snyk-container-9.png)

* Dockerfile を選択してスキャン結果を確認します。レジストリからのコンテナイメージのスキャン結果と中身は似ていますが、ここでは Dockerfile をスキャンしていることに注意してください。

Dockerfile プロジェクトでは、Dockerfile のメタデータとそこで使われているベースイメージについての情報が表示されます。ベースイメージが [Docker 公式イメージ](https://docs.docker.com/docker-hub/official_images/)であれば、スキャン結果にはベースイメージの更新の推奨が含まれ、検出された脆弱性の一部を更新を通じて修正できます。

## Step 6 Pull Request によるベースイメージの更新

Here we will go ahead and fix our Dockerfile using the "**Open a Fix PR**" button as follows:

* Click on "**Open a Fix PR**" for the base image "**node:16.6.0-slim**" as per below

![alt tag](https://i.ibb.co/5kY26FR/snyk-container-13.png)

* Click on "**Open a Fix PR**" on the resulting page as shown below

![alt tag](https://i.ibb.co/C0tn01C/snyk-container-14.png)

* A PR is then created as show below. "**Files Changed**" will show you what it's updating in the Dockerfile itself

![alt tag](https://i.ibb.co/py4GdJS/snyk-container-15.png)

* Click on "**Merge Pull Request**" button as shown below

![alt tag](https://i.ibb.co/hCwDCFP/snyk-container-16.png)

* Return to the projects dashboard and you will see a new scan has occurred automatically and now our Dockerfile shows much less issues than previously. Of course until we build a new container and add it to the registry the container itself will still have the old base image in place.

![alt tag](https://i.ibb.co/pbqmR1v/snyk-container-17.png)

## Step 7 Snyk CLI を用いたコンテナイメージのスキャン

The Snyk CLI can run a container test on containers sitting in a registry and even your local docker deamon if you like. All the Snyk CLI needs is access to the registry itself which is for public Docker Hub images only requires a "docker login" to achieve that. The following examples show how to use the Snyk CLI to issue a container test.

* Before we get started please make sure you have setup the Snyk CLI. There are various install options as per the links below. Using the prebuilt binaries means you don't have to install NPM to install the Snyk CLI.

1. Install Page - https://support.snyk.io/hc/en-us/articles/360003812538-Install-the-Snyk-CLI
1. Prebuilt Binaries - https://github.com/snyk/snyk/releases

_Note: Make sure you have the following version installed or later_

```bash
$ snyk --version
1.675.0
```

* Authorize the snyk CLI with your account as follows

```bash
$ snyk auth

Now redirecting you to our auth page, go ahead and log in,
and once the auth is complete, return to this prompt and you'll
be ready to start using snyk.

If you can't wait use this url:
https://snyk.io/login?token=ff75a099-4a9f-4b3d-b75c-bf9847672e9c&utm_medium=cli&utm_source=cli&utm_campaign=cli&os=darwin&docker=false

Your account has been authenticated. Snyk is now ready to be used.
```

_Note: If you are having trouble authenticating via a browser with the Snyk App you can setup authentication using the API token as shown below
[Authenticate using your API token](https://support.snyk.io/hc/en-us/articles/360004008258-Authenticate-the-CLI-with-your-account#UUID-4f46843c-174d-f448-cadf-893cfd7dd858_section-idm4557419555668831541902780562)_

_Note: Testing container images through the CLI performs the following steps, so it can take a few minutes on the first scan [Test images with the Snyk Container CLI](https://support.snyk.io/hc/en-us/articles/360003946917-Test-images-with-the-Snyk-Container-CLI)_

1. Downloads the image if it’s not already available locally in your Docker daemon
2. Determines the software installed in the image
3. Sends that bill of materials to the Snyk Service
4. Returns a list of the vulnerabilities in your image

* You have already built "docker-goof" so go ahead and test that as shown below, please use your dockerhub account username rather than "**pasapples**"

```bash
$ snyk container test pasapples/docker-goof:latest

...

Tested 412 dependencies for known issues, found 889 issues.

Base Image  Vulnerabilities  Severity
node:14.1   889              45 critical, 186 high, 196 medium, 462 low

Recommendations for base image upgrade:

Minor upgrades
Base Image  Vulnerabilities  Severity
node:14.17  530              9 critical, 46 high, 40 medium, 435 low

Major upgrades
Base Image   Vulnerabilities  Severity
node:16.5.0  344              3 critical, 31 high, 49 medium, 261 low

Alternative image types
Base Image                Vulnerabilities  Severity
node:16.6.0-slim          60               2 critical, 8 high, 5 medium, 45 low
node:16.6.1-buster-slim   60               2 critical, 8 high, 5 medium, 45 low
node:16.6.0-buster        343              3 critical, 30 high, 49 medium, 261 low
node:16.6.0-stretch-slim  78               6 critical, 11 high, 8 medium, 53 low
```

* The following container test is for a Spring Boot application 

```bash
$ snyk container test pasapples/springbootemployee:cnb

....

Organization:      pas.apicella-41p
Package manager:   deb
Project name:      docker-image|pasapples/springbootemployee
Docker image:      pasapples/springbootemployee:cnb
Platform:          linux/amd64
Base image:        ubuntu:bionic-20210325
Licenses:          enabled

Tested 97 dependencies for known issues, found 32 issues.

Base Image              Vulnerabilities  Severity
ubuntu:bionic-20210325  31               0 critical, 1 high, 6 medium, 24 low

Recommendations for base image upgrade:

Minor upgrades
Base Image              Vulnerabilities  Severity
ubuntu:bionic-20210723  25               0 critical, 0 high, 3 medium, 22 low

Major upgrades
Base Image    Vulnerabilities  Severity
ubuntu:20.04  15               0 critical, 0 high, 0 medium, 15 low
```

* There is also a Distroless version if you would like to try with that

```bash
$ snyk container test pasapples/spring-crud-thymeleaf-demo:distroless

...

Organization:      pas.apicella-41p
Package manager:   deb
Project name:      docker-image|pasapples/spring-crud-thymeleaf-demo
Docker image:      pasapples/spring-crud-thymeleaf-demo:distroless
Platform:          linux/amd64
Licenses:          enabled

Tested 20 dependencies for known issues, found 38 issues.
```

* The CLI also allows us to report vulnerabilities of provided level or higher. Now let's go ahead and set that to HIGH using "**--severity-threshold=high**". Only issues tagged as HIGH or CRITICAL will appear on this test run.

```shell
$ snyk container test --severity-threshold=high pasapples/springbootemployee:cnb

Testing pasapples/springbootemployee:cnb...

✗ High severity vulnerability found in systemd/libsystemd0
  Description: Allocation of Resources Without Limits or Throttling
  Info: https://snyk.io/vuln/SNYK-UBUNTU1804-SYSTEMD-1320128
  Introduced through: systemd/libsystemd0@237-3ubuntu10.46, apt/libapt-pkg5.0@1.6.13, procps/libprocps6@2:3.3.12-3ubuntu1.2, util-linux/bsdutils@1:2.31.1-0.4ubuntu3.7, util-linux/mount@2.31.1-0.4ubuntu3.7, systemd/libudev1@237-3ubuntu10.46
  From: systemd/libsystemd0@237-3ubuntu10.46
  From: apt/libapt-pkg5.0@1.6.13 > systemd/libsystemd0@237-3ubuntu10.46
  From: procps/libprocps6@2:3.3.12-3ubuntu1.2 > systemd/libsystemd0@237-3ubuntu10.46
  and 5 more...
  Fixed in: 237-3ubuntu10.49

✗ High severity vulnerability found in openssl
  Description: Buffer Overflow
  Info: https://snyk.io/vuln/SNYK-UBUNTU1804-OPENSSL-1569474
  Introduced through: openssl@1.1.1-1ubuntu2.1~18.04.9, ca-certificates@20210119~18.04.1, openssl/libssl1.1@1.1.1-1ubuntu2.1~18.04.9
  From: openssl@1.1.1-1ubuntu2.1~18.04.9
  From: ca-certificates@20210119~18.04.1 > openssl@1.1.1-1ubuntu2.1~18.04.9
  From: openssl/libssl1.1@1.1.1-1ubuntu2.1~18.04.9
  and 1 more...
  Fixed in: 1.1.1-1ubuntu2.1~18.04.13



Organization:      pas.apicella-41p
Package manager:   deb
Project name:      docker-image|pasapples/springbootemployee
Docker image:      pasapples/springbootemployee:cnb
Platform:          linux/amd64
Base image:        ubuntu:bionic-20210325
Licenses:          enabled

Tested 97 dependencies for known issues, found 2 issues.

Base Image              Vulnerabilities  Severity
ubuntu:bionic-20210325  31               0 critical, 1 high, 6 medium, 24 low

Recommendations for base image upgrade:

Minor upgrades
Base Image              Vulnerabilities  Severity
ubuntu:bionic-20210723  25               0 critical, 0 high, 3 medium, 22 low

Major upgrades
Base Image    Vulnerabilities  Severity
ubuntu:20.04  15               0 critical, 0 high, 0 medium, 15 low

```

The severity threshold allows you to break a build in the event of this threshold being meet using the exit code of the command. 

```
Possible exit codes and their meaning:

0: success, no vulns found
1: action_needed, vulns found
2: failure, try to re-run the command
3: failure, no supported projects detected
```

* Finally, can monitor container images using the "**snyk container monitor**" command as shown below, please perform this step now using a different container image this time

```bash
$ snyk container monitor pasapples/spring-crud-thymeleaf-demo:latest --project-name=spring-crud-thymeleaf-demo-container

Monitoring pasapples/spring-crud-thymeleaf-demo:latest (spring-crud-thymeleaf-demo-container)...

Explore this snapshot at https://app.snyk.io/org/workshops-admin-org/project/1cce2457-a6ac-4f09-8465-9ca596a966fa/history/a253804a-bdb2-4ed1-a7d6-54a321e7f725

Notifications about newly disclosed issues related to these dependencies will be emailed to you.
```

![alt tag](https://i.ibb.co/vY63PXR/snyk-container-12.png)

* Return to the Snyk App and this time you will see the container image 

Thanks for attending and completing this workshop

![alt tag](https://i.ibb.co/7tnp1B6/snyk-logo.png)

<hr />
Pas Apicella [pas at snyk.io] is an Solution Engineer at Snyk APJ
