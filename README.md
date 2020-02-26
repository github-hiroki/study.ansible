# study.ansible
Ansibleの勉強。

## 環境
`macOS(10.15.3)`で実行しています。

## インストール方法
~~~console
# pipenv install
~~~

## 実行方法
~~~console
# pipenv shell
(study.ansible)# ansible --version
~~~
~~~shell-session
ansible 2.9.5
...
~~~
`exit`でシェルを抜けることができます。
~~~console
(study.ansible)$ exit
~~~

## dockerコンテナの作成
わざわざVMを作成するのが面倒、また、手軽に初期化したいので検証用としてdockerコンテナを使用します。
dockerのインストールは`Docker Desktop for Mac`にて行いました。

### ビルド
~~~console
# docker build --tag python:ansible ./container
~~~

### 実行
~~~console
# docker run -d --rm -p 22:22 python:ansible
~~~

### ssh接続
以下でssh接続することができます。
~~~console
# ssh -i ./container/id_rsa root@localhost
~~~
ただし、コンテナは起動するたびに別物となりますので、`~/.ssh/known_hosts`へ書き込みを抑制するために`-o UserKnownHostsFile=/dev/null`をつけましょう。
また、ログイン時のチェックを行わないようにするために`-o StrictHostKeyChecking=no`もつけちゃいましょう。
~~~console
# ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -i ./container/id_rsa root@localhost
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

## 感想
Ansibleの勉強のためのリポジトリでしたが、他の開発メンバーへの共有のためにハンズオンという形でまとめていきたいと思います。
初め触った際の感想は「とっても便利」でした。
私自身はサーバー構築する機会はあまりないですが、メンテナンス時のシェルキックに使ってみたいなぁ、などと触りながら思っていた気がします。
