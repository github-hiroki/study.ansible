# Ansible入門

`Ansible` という名前は聞いたことある、どのようなものか何となく知ってはいる、だけれど使ったことない、という人向けのハンズオンです。
リモート環境に `docker` を使用していますので、`docker` を触ったことがない方には抵抗あるかもしれません。

`docker` ハンズオンはたくさんあると思いますが、いつか作成できたら...いいなぁ（遠い目）

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
# docker run -h dc01 --name dc01 -d --rm -p 10022:22 python:ansible
# docker run -h dc02 --name dc02 -d --rm -p 20022:22 python:ansible
# docker network create study.ansible
# docker run -h dc03 --name dc03 --network study.ansible -d --rm python:3-alpine tail -f /dev/null
# docker run --privileged -h dc04 --name dc04 --network study.ansible -d --rm centos:centos7 /sbin/init
# docker run -h dc05 --name dc05 --network study.ansible -d --rm python:2-buster tail -f /dev/null
~~~

ホスト名はそれぞれ `dc01`,`dc02`,`dc03`,`dc04`,`dc05` としています。

`dc03`,`dc04` は [06.docker connection plugin](hands-on/06/README.md) 以降、`dc05` は [09.whenディレクティブ](hands-on/09/README.md) 以降で使用しています。
こちらのコンテナには ssh による接続はできません。

コンテナを停止する場合は以下を実行してください。起動時に `--rm` オプションを指定していますので、停止すれば削除されます。

~~~console
# docker stop dc01 dc02 dc03 dc04 dc05
# docker network rm study.ansible
~~~

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

これらのオプションがansible実行時に指定されるように、各セッション(`hands-on/01`〜`hands-on/05`)に `ansible.cfg` を用意してあります。

~~~ini
[ssh_connection]
ssh_args = -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -i ../../container/id_rsa
~~~

## ハンズオン

### [01.ansibleコマンド](hands-on/01/README.md)

### [02.インベントリ変数](hands-on/02/README.md)

### [03.レジスタ変数](hands-on/03/README.md)

### [04.ファクト変数の参照とif文](hands-on/04/README.md)

### [05.インベントリに複数サーバーを定義](hands-on/05/README.md)

### [06.docker connection plugin](hands-on/06/README.md)

### [07.定義済み変数とfor文](hands-on/07/README.md)

### [08.変数フィルタ](hands-on/08/README.md)

### [09.whenディレクティブ](hands-on/09/README.md)

### [10.loopディレクティブ](hands-on/10/README.md)

### [11.blockディレクティブ](hands-on/11/README.md)

## 感想

Ansibleの勉強のためのリポジトリでしたが、他の開発メンバーへの共有のためにハンズオンという形でまとめていきたいと思います。

初め触った際の感想は「とっても便利」でした。
私自身はサーバー構築する機会はあまりないですが、メンテナンス時のシェルキックに使ってみたいなぁ、などと触りながら思っていた気がします。

学習していく中で、やろうと思えばサーバーへの操作を何でも行えて便利、など考えてしまいましたが、冪等性の担保を常に頭にいれておかないと痛い目に会いそうです。気をつけねば。
