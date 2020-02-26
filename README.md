# study.ansible

Ansibleの勉強。

## 環境

`macOS(10.15.3)`で実行しています。

## 事前準備

事前準備として、作業PCとして以下の環境を用意してください。

- pyenvのインストール
- pipenvのインストール
- dockerのビルド/実行環境

## Ansibleのインストール

`Pipfile`を用意してあります。以下を実行すると`ansible`を実行できる仮想環境が作成されます。

~~~console
# pipenv install
~~~

## 仮想環境への切り替え

~~~console
# pipenv shell
(study.ansible)# ansible --version
~~~

~~~shell-session
ansible 2.9.5
...
~~~

これで`ansible`を実行することができるようになりました。また、終了時は `exit` で抜けることができます。

~~~console
(study.ansible)$ exit
~~~

## dockerコンテナの作成

わざわざVMを作成するのが面ど..げふんげふん、手軽に作成/初期化したいので動作確認用にdockerコンテナを使用します。
`Docker Desktop for Mac` をインストールした環境にて行っていますが、`docker`コンテナのビルド/実行ができる環境であれば問題ありません。

### ビルド

~~~console
# docker build --tag python:ansible ./container
~~~

### 実行

~~~console
# docker run -h dc01 -d --rm -p 10022:22 python:ansible
# docker run -h dc02 -d --rm -p 20022:22 python:ansible
~~~

ホスト名はそれぞれ `dc01`,`dc02` としています。

### ssh接続

以下でssh接続することができます。

~~~console
# ssh -i ./container/id_rsa root@localhost -p 10022
~~~

ただし、コンテナは起動するたびに別物となりますので、`~/.ssh/known_hosts`へ書き込みを抑制するために`-o UserKnownHostsFile=/dev/null`をつけましょう。
また、ログイン時のチェックを行わないようにするために`-o StrictHostKeyChecking=no`もつけちゃいましょう。

~~~console
# ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -i ./container/id_rsa root@localhost -p 10022
dc01:~#
~~~

これらのオプションがansible実行時に指定されるように、各セッション(`hands-on/XX`)に `ansible.cfg` を用意してあります。

~~~ini
[ssh_connection]
ssh_args = -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -i ../../container/id_rsa
~~~

## ハンズオン

### [01.入門](hands-on/01/README.md)

### [02.インベントリ変数](hands-on/02/README.md)

### [03.レジスタ変数](hands-on/03/README.md)

### [04.ファクト変数の参照とif文](hands-on/04/README.md)

### [05.インベントリに複数サーバーを定義](hands-on/05/README.md)

## 感想

Ansibleの勉強のためのリポジトリでしたが、他の開発メンバーへの共有のためにハンズオンという形でまとめていきたいと思います。
初め触った際の感想は「とっても便利」でした。
私自身はサーバー構築する機会はあまりないですが、メンテナンス時のシェルキックに使ってみたいなぁ、などと触りながら思っていた気がします。
