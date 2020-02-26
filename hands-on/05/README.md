# 05.インベントリに複数サーバーを定義

## 実行環境

~~~console
# pipenv shell
# cd hands-on/05
~~~

## インベントリの編集

2つのサーバーへアクセスするように見せかけて、どちらも `localhost` (`docker`コンテナ) ですので、`ansible_host` 変数を使用して設定します。

~~~console
# vi inventory/inventory.ini
~~~

~~~ini
[docker]
dc01
dc02
~~~

~~~console
# vi inventory/group_vars/all/login.yml
~~~

~~~yml
---
ansible_ssh_user: root
~~~

~~~console
# mkdir -p inventory/host_vars/dc01
# vi inventory/host_vars/dc01/login.yml
~~~

~~~yml
---
ansible_host: localhost
ansible_ssh_port: 10022
~~~

~~~console
# mkdir -p inventory/host_vars/dc02
# vi inventory/host_vars/dc02/login.yml
~~~

~~~yml
---
ansible_host: localhost
ansible_ssh_port: 20022
~~~

これで設定変更は完了です。

[04.ファクト変数の参照とif文](../04/README.md) と同様の`Playbook` を実行してみましょう。

~~~console
# ansible-playbook -i inventory/inventory.ini playbooks/playbook.yml
~~~

~~~shell-session
PLAY [docker] **************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************
ok: [dc01]
ok: [dc02]

TASK [Get list on any directory] *******************************************************************************************************************
changed: [dc01]
changed: [dc02]

TASK [Debug registered variable] *******************************************************************************************************************
ok: [dc01] => {
    "result": {
        "changed": true,
        "cmd": " ls /root\n",
        "delta": "0:00:00.005000",
        "end": "2020-02-26 09:21:59.669919",
        "failed": false,
        "rc": 0,
        "start": "2020-02-26 09:21:59.664919",
        "stderr": "",
        "stderr_lines": [],
        "stdout": "sample.txt\nwork",
        "stdout_lines": [
            "sample.txt",
            "work"
        ]
    }
}
ok: [dc02] => {
    "result": {
        "changed": true,
        "cmd": " ls /\n",
        "delta": "0:00:00.005656",
        "end": "2020-02-26 09:21:59.684211",
        "failed": false,
        "rc": 0,
        "start": "2020-02-26 09:21:59.678555",
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

PLAY RECAP *****************************************************************************************************************************************
dc01                       : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
dc02                       : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
~~~

それぞれのサーバーへ実行することができました。
