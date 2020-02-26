# study.ansible
Ansibleのお勉強。

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
# ssh -i ./id_rsa root@localhost
~~~
ただし、コンテナは起動するたびに別物となりますので、`~/.ssh/known_hosts`へ書き込みを抑制するために`-o UserKnownHostsFile=/dev/null`をつけましょう。
また、ログイン時のチェックを行わないようにするために`-o StrictHostKeyChecking=no`もつけちゃいましょう。
~~~console
# ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -i ./container/id_rsa root@localhost
~~~
これらのオプションがansible実行時に指定されるように`ansible.cfg`に設定しておきます。
~~~console
# vi ansible.cfg
~~~
~~~ini
[ssh_connection]
ssh_args = -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -i ./container/id_rsa
~~~

## インベントリファイル
作成したdockerコンテナをターゲットノードにするためのインベントリファイルを作成します。
~~~console
# mkdir inventory
# vi inventory/inventory.yml
~~~
~~~yaml
all:
  children:
    docker:
      hosts:
        localhost:
  vars:
    ansible_python_interpreter: /usr/local/bin/python
    ansible_ssh_user: root
~~~
`inventory/inventory.yml`にコンテナ内のpython実行パスを`ansible_python_interpreter`で定義しています。また、ターゲットノードのユーザーをrootにするために`ansible_ssh_user`を定義しています。

## ansibleコマンド
### pingを実行してみる
~~~console
# pipenv shell
# ansible -i inventory/inventory.yml docker -m ping
~~~
~~~shell-session
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
~~~

### ファイルを作成してみる
~~~console
# ansible -i inventory/inventory.yml docker -m file -a 'path=/root/test.txt state=touch mode=0644'
~~~
~~~shell-session
localhost | CHANGED => {
    "changed": true,
    "dest": "/root/test.txt",
    "gid": 0,
    "group": "root",
    "mode": "0644",
    "owner": "root",
    "size": 0,
    "state": "file",
    "uid": 0
}
~~~

## ansible-playbookコマンド
### ディレクトリの生成とファイルコピー
~~~console
# vi playbook/sample01.yml
~~~
~~~yaml
- hosts: docker
  tasks:
  - name: create directory
    file:
      path: /root/work
      state: directory
      owner: root
      mode: 0755

  - name: copy file
    copy:
      src: sample01.yml
      dest: /root/work/sample01.yml
      owner: root
      mode: 0644
~~~
~~~console
# ansible-playbook -i inventory/inventory.yml playbooks/sample01.yml
~~~
~~~shell-session
PLAY [docker] *****************************************************************************

TASK [Gathering Facts] ********************************************************************
ok: [localhost]

TASK [create directory] *******************************************************************
changed: [localhost]

TASK [copy file] **************************************************************************
changed: [localhost]

PLAY RECAP ********************************************************************************
localhost                  : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
~~~

## 感想
とっても便利でした。
私自身はサーバー構築する機会はあまりないですが、メンテナンス時のシェルキックに使ってみたいなぁ、などと触りながら思いました。
