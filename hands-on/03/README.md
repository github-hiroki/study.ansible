# 03.レジスタ変数

## 概要

`ansible`では実行したコマンドの結果を任意の変数に入れることができます。

変数に格納するのに必要となるのが`register`コマンドです。

実行するコマンドと `register: 任意の変数名` を併記することで、以降のコマンドで変数を使用することができます。

例えば、特定のファイルの存在有無をチェックして結果を変数に格納し、以降の処理でファイルが存在しない場合にのみコマンドを実行することができます。

コマンドの実行時に変数により処理を切り替えるには [09.whenディレクティブ](../09/README.md) を使用することになります。

## 実行環境

~~~console
# pipenv shell
# cd hands-on/03
~~~

## レジスタ変数

レジスタ変数を使用したプレイブックを用意します。

~~~console
# vi playbooks/playbook.yml
~~~

~~~yml
---
- hosts: docker
  tasks:
  - name: Get list on root directory
    shell: ls /
    register: result

  - name: Debug registered variable
    debug:
      var: result
~~~

`playbooks/playbook.yml`内の `register` にてレジスタ変数を定義します。

`debug` モジュールを使用するとレジスタ変数の確認を行うこともできます。

~~~console
# ansible-playbook -i inventory/inventory.ini playbooks/playbook.yml
~~~

~~~shell-session
PLAY [docker] ******************************************************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]

TASK [Get list on root directory] **********************************************
changed: [localhost]

TASK [Debug registered variable] ***********************************************
ok: [localhost] => {
    "result": {
        "changed": true,
        "cmd": "ls /",
        "delta": "0:00:00.003003",
        "end": "2020-02-26 07:30:45.928793",
        "failed": false,
        "rc": 0,
        "start": "2020-02-26 07:30:45.925790",
        "stderr": "",
        "stderr_lines": [],
        "stdout": "bin\ndev\netc\nhome\nlib\nmedia\nmnt\nopt\nproc\nroot\nrun\nsbin\nsrv\nsys\ntmp\nusr\nvar",
        "stdout_lines": [
            "bin",
            "dev",
            "etc",
            "home",
            "lib",
            "media",
            "mnt",
            "opt",
            "proc",
            "root",
            "run",
            "sbin",
            "srv",
            "sys",
            "tmp",
            "usr",
            "var"
        ]
    }
}

PLAY RECAP *********************************************************************
localhost                  : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
~~~
