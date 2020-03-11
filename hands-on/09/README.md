# 09.whenディレクティブ

## 実行環境

~~~console
# pipenv shell
# cd hands-on/09
~~~

## インベントリの編集

`when`ディレクティブを試行するにあたって、インベントリに `dc05` を追加します。

~~~console
# vi inventory/inventory.ini
~~~

~~~yaml
[docker]
dc03
dc04
dc05

[alpine]
dc03

[centos]
dc04

[debian]
dc05
~~~

`dc05`(`python:2-buster`) コンテナには python モジュールが2つ存在しているので明示的に実行パスを指定します。

~~~console
# vi inventory/host_vars/dc05/python.yml
~~~

~~~yaml
---
ansible_python_interpreter: /usr/bin/python
~~~

## プレイブックの編集

`when` ディレクティブを使用して `postfix` をインストールするプレイブックを作成してみましょう。

~~~console
# vi playbooks/playbook.yml
~~~

~~~yaml
---
- hosts: docker
  connection: docker
  tasks:

  - name: Install the postfix in RedHat
    yum:
      name: postfix
      state: present
    when: ansible_os_family == "RedHat"

  - name: Install the postfix in Alpine
    apk:
      name: postfix
      state: present
    when: ansible_os_family == "Alpine"

  - name: Install the postfix in Debian
    apt:
      name: postfix
      state: present
    when: ansible_os_family == "Debian"
~~~

それでは `Playbook` を実行してみましょう。

~~~console
# ansible-playbook -i inventory/inventory.ini playbooks/playbook.yml
~~~

~~~shell-session
PLAY [docker] ******************************************************************************

TASK [Gathering Facts] *********************************************************************
ok: [dc03]
ok: [dc05]
ok: [dc04]

TASK [Install the postfix in RedHat] *******************************************************
skipping: [dc03]
skipping: [dc05]
changed: [dc04]

TASK [Install the postfix in Alpine] *******************************************************
skipping: [dc04]
skipping: [dc05]
changed: [dc03]

TASK [Install the postfix in Debian] *******************************************************
skipping: [dc03]
skipping: [dc04]
[WARNING]: Updating cache and auto-installing missing dependency: python-apt
changed: [dc05]

PLAY RECAP *********************************************************************************
dc03                       : ok=2    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
dc04                       : ok=2    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
dc05                       : ok=2    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
~~~

無事にインストールできました。
再度実行してみると `changed` -> `ok` になっています。

~~~shell-session
PLAY [docker] ******************************************************************************

TASK [Gathering Facts] *********************************************************************
ok: [dc03]
ok: [dc05]
ok: [dc04]

TASK [Install the postfix in RedHat] *******************************************************
skipping: [dc03]
skipping: [dc05]
ok: [dc04]

TASK [Install the postfix in Alpine] *******************************************************
skipping: [dc04]
skipping: [dc05]
ok: [dc03]

TASK [Install the postfix in Debian] *******************************************************
skipping: [dc03]
skipping: [dc04]
ok: [dc05]

PLAY RECAP *********************************************************************************
dc03                       : ok=2    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
dc04                       : ok=2    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
dc05                       : ok=2    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
~~~
