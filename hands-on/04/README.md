# 04.ファクト変数の参照とif文

## 実行環境

~~~console
# pipenv shell
# cd hands-on/04
~~~

## ファクト変数

ファクト変数はこちらを入力することで取得できます。

~~~console
# ansible -i inventory/inventory.ini localhost -m setup
~~~

~~~shell-session
...長いので省略
~~~

ファクト変数 `ansible_hostname` を使用して `if` 文を含むプレイブックを作成してみます。

~~~console
# vi playbooks/playbook.yml
~~~

~~~yml
---
- hosts: docker
  tasks:
  - name: Get list on any directory
    shell: |
      {% if   ansible_hostname == 'dc02' %} ls /
      {% elif ansible_hostname == 'dc01' %} ls /root
      {% else %} ls /etc
      {% endif %}
    register: result

  - name: Debug registered variable
    debug:
      var: result
~~~

現時点では `docker` グループのサーバーは `dc01` となっているはずですので、以下のような実行結果となります。

~~~console
# ansible-playbook -i inventory/inventory.ini playbooks/playbook.yml
~~~

~~~shell-session
PLAY [docker] *****************************************************************************

TASK [Gathering Facts] ********************************************************************
ok: [localhost]

TASK [Get list on any directory] **********************************************************
changed: [localhost]

TASK [Debug registered variable] **********************************************************
ok: [localhost] => {
    "result": {
        "changed": true,
        "cmd": " ls /root\n",
        "delta": "0:00:00.003464",
        "end": "2020-02-26 08:50:38.818317",
        "failed": false,
        "rc": 0,
        "start": "2020-02-26 08:50:38.814853",
        "stderr": "",
        "stderr_lines": [],
        "stdout": "sample.txt\nwork",
        "stdout_lines": [
            "sample.txt",
            "work"
        ]
    }
}

PLAY RECAP ********************************************************************************
localhost                  : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
~~~
