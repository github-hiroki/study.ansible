# 10.loopディレクティブ

## 実行環境

~~~console
# pipenv shell
# cd hands-on/10
~~~

## インベントリ

[09.whenディレクティブ](../09/README.md) のインベントリをそのまま使用します。

## プレイブックの編集

`loop` ディレクティブを使用して複数のユーザーを作成するプレイブックを作成してみましょう。

~~~console
# vi playbooks/playbook.yml
~~~

~~~yaml
---
- hosts: docker
  connection: docker
  tasks:

  - name: Add SourcePod group
    group:
      name: sp
      state: present

  - name: Add developers
    user:
      name: "{{ item.name }}"
      groups: "{{ item.groups }}"
      state: present
    loop:
      - { name: 'hiroki', groups: 'sp' }
      - { name: 'guest', groups: 'sp' }
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

TASK [Add SourcePod group] *****************************************************************
changed: [dc03]
changed: [dc04]
changed: [dc05]

TASK [Add developers] ***********************************************************************
changed: [dc03] => (item={'name': 'hiroki', 'groups': 'sp'})
changed: [dc05] => (item={'name': 'hiroki', 'groups': 'sp'})
changed: [dc04] => (item={'name': 'hiroki', 'groups': 'sp'})
changed: [dc03] => (item={'name': 'guest', 'groups': 'sp'})
changed: [dc05] => (item={'name': 'guest', 'groups': 'sp'})
changed: [dc04] => (item={'name': 'guest', 'groups': 'sp'})

PLAY RECAP *********************************************************************************
dc03                       : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
dc04                       : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
dc05                       : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
~~~

無事に作成されました。
再度実行してみると `changed` -> `ok` になっていることを確認できます。

~~~shell-session
PLAY [docker] ******************************************************************************

TASK [Gathering Facts] *********************************************************************
ok: [dc03]
ok: [dc05]
ok: [dc04]

TASK [Add SourcePod group] *****************************************************************
ok: [dc05]
ok: [dc04]
ok: [dc03]

TASK [Add developers] **********************************************************************
ok: [dc03] => (item={'name': 'hiroki', 'groups': 'sp'})
ok: [dc04] => (item={'name': 'hiroki', 'groups': 'sp'})
ok: [dc05] => (item={'name': 'hiroki', 'groups': 'sp'})
ok: [dc03] => (item={'name': 'guest', 'groups': 'sp'})
ok: [dc04] => (item={'name': 'guest', 'groups': 'sp'})
ok: [dc05] => (item={'name': 'guest', 'groups': 'sp'})

PLAY RECAP *********************************************************************************
dc03                       : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
dc04                       : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
dc05                       : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
~~~
