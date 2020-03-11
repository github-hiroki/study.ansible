# 01.ansibleコマンド

## 実行環境

~~~console
# pipenv shell
# cd hands-on/01
~~~

## インベントリファイル

作成したdockerコンテナをターゲットノードにするためのインベントリファイルを作成します。

~~~console
# mkdir inventory
# vi inventory/inventory.ini
~~~

~~~ini
[docker]
localhost

[all:vars]
ansible_python_interpreter=/usr/local/bin/python
ansible_ssh_user=root
ansible_ssh_port=10022
~~~

`inventory/inventory.ini`にコンテナ内のpython実行パスを`ansible_python_interpreter`で定義しています。また、ターゲットノードのユーザーをrootにするために`ansible_ssh_user`を定義しています。

## ansibleコマンド

### pingを実行してみる

~~~console
# ansible -i inventory/inventory.ini docker -m ping
~~~

~~~shell-session
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
~~~

### ファイルを作成してみる

~~~console
# ansible -i inventory/inventory.ini docker -m file -a 'path=/root/sample.txt state=touch mode=0644'
~~~

~~~shell-session
localhost | CHANGED => {
    "changed": true,
    "dest": "/root/sample.txt",
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

`ansible` コマンドでは一つの処理を実行していましたが、サーバー構築するためには何度も `ansible` コマンドを実行しなければなりません。これを解決するのが `ansible-playbook` コマンドです。プレイブック（サーバー構築でいえば構築仕様書ようなもの）を用意することで一度の実行で全ての処理を行えます。

### ディレクトリの生成とファイルコピー

~~~console
# mkdir playbooks
# vi playbooks/playbook.yml
~~~

~~~yaml
---
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
      src: playbook.yml
      dest: /root/work/playbook.yml
      owner: root
      mode: 0644
~~~

~~~console
# ansible-playbook -i inventory/inventory.ini playbooks/playbook.yml
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
