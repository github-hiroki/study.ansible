# 06.docker connection plugin

## 実行環境

~~~console
# pipenv shell
# cd hands-on/06
~~~

## インベントリの編集

ssh接続できなくとも docker コンテナへ接続するためのプラグインが存在するという話を聞いたので試してみます。
インベントリをsshできない `dc03`,`dc04` コンテナへアクセスするための情報に書き換えます。

~~~console
# vi inventory/inventory.ini
~~~

~~~ini
[docker]
dc03
dc04

[alpine]
dc03

[centos]
dc04
~~~

グループは上記のように複数に所属させることができます。

上記以外にも `all` というグループに所属しています。

`python` の実行パスをホスト毎に指定します。

~~~console
# vi inventory/host_vars/dc03/python.yml
~~~

~~~yml
---
ansible_python_interpreter: /usr/local/bin/python
~~~

dc04 は `Python2` のため設定は実行パスの設定は不要ですが、centos コンテナのログインユーザーが `root` ではありませんので、インベントリにて設定を加えます。

~~~console
# vi inventory/host_vars/dc04/login.yml
~~~

~~~yaml
---
ansible_user: root
~~~

前章にて ssh 接続するために使用していた以下のログイン設定は不要になります。

~~~console
# rm -f inventory/group_vars/all/login.yml
# rm -f inventory/group_vars/all/python.yml
# rm -rf inventory/host_vars/dc01
# rm -rf inventory/host_vars/dc02
~~~

同様に前章まで用意していた `ansible.cfg` も不要となります。

~~~console
# rm -f ansible.cfg
~~~

これだけでは単に接続対象のホスト名を変更して、接続に必要な設定を削除しただけです。

接続するためにはプレイブックの編集が必要となります。

## プレイブックの編集

接続するためにはプレイブックに一行追加する必要があります。

~~~console
# vi playbooks/playbook.yml
~~~

~~~yaml
---
- hosts: docker
  connection: docker
  tasks:
  - name: Get list on any directory
    shell: |
      {% if   ansible_hostname == 'dc03' %} ls /
      {% elif ansible_hostname == 'dc04' %} ls /root
      {% else %} ls /etc
      {% endif %}
    register: result

  - name: Debug registered variable
    debug:
      var: result
~~~

前章との差分を比較すると一行追加されているのが分かります。

~~~console
diff -u ../05/playbooks/playbook.yml playbooks/playbook.yml
~~~

~~~diff
 ---
 - hosts: docker
+  connection: docker
   tasks:
   - name: Get list on any directory
     shell: |
-      {% if   ansible_hostname == 'dc02' %} ls /
-      {% elif ansible_hostname == 'dc01' %} ls /root
+      {% if   ansible_hostname == 'dc03' %} ls /
+      {% elif ansible_hostname == 'dc04' %} ls /root
       {% else %} ls /etc
       {% endif %}
     register: result
~~~

これで設定変更は完了です。`Playbook` を実行してみましょう。

~~~console
# ansible-playbook -i inventory/inventory.ini playbooks/playbook.yml
~~~

~~~shell-session
PLAY [docker] ******************************************************************************

TASK [Gathering Facts] *********************************************************************
ok: [dc03]
ok: [dc04]

TASK [Get list on any directory] ***********************************************************
changed: [dc03]
changed: [dc04]

TASK [Debug registered variable] ***********************************************************
ok: [dc03] => {
    "result": {
        "changed": true,
        "cmd": " ls /\n",
        "delta": "0:00:00.004004",
        "end": "2020-03-11 03:01:44.445969",
        "failed": false,
        "rc": 0,
        "start": "2020-03-11 03:01:44.441965",
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
ok: [dc04] => {
    "result": {
        "changed": true,
        "cmd": " ls /root\n",
        "delta": "0:00:00.010015",
        "end": "2020-03-11 03:01:44.464266",
        "failed": false,
        "rc": 0,
        "start": "2020-03-11 03:01:44.454251",
        "stderr": "",
        "stderr_lines": [],
        "stdout": "anaconda-ks.cfg",
        "stdout_lines": [
            "anaconda-ks.cfg"
        ]
    }
}

PLAY RECAP *********************************************************************************
dc03                       : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
dc04                       : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
~~~

`ssh`接続できない`docker`コンテナに対して実行することができました。
