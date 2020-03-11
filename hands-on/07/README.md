# 07.定義済み変数とfor文

## 実行環境

~~~console
# pipenv shell
# cd hands-on/07
~~~

## インベントリ

[06.docker connection plugin](../06/README.md) のインベントリをそのまま使用します。

## プレイブックの編集

`for`文を使用したプレイブックを作成してみましょう。

~~~console
# vi playbooks/playbook.yml
~~~

~~~yaml
---
- hosts: docker
  connection: docker
  tasks:
  - name: Get linux distribution
    shell: |
      {% for name in groups['all'] %}
        {% if ansible_hostname == name %}
          echo {{ name }} {{ hostvars[name]['ansible_facts']['distribution'] }} {{ hostvars[name]['ansible_facts']['distribution_version'] }}
        {% endif %}
      {% endfor %}
    register: result

  - name: Debug registered variable
    debug:
      var: result.stdout
~~~

実行対象(`ansible_hostname`)のディストリビュージョンのみを表示するようにしているため、`for`文の意味はなくなってしまっていますが。。
`for`文を使用せずにもっとシンプルに書けます。

~~~yaml
---
- hosts: docker
  connection: docker
  tasks:
  - name: Get linux distribution
    shell: |
        echo {{ ansible_hostname }} {{ hostvars[ansible_hostname]['ansible_facts']['distribution'] }} {{ hostvars[ansible_hostname]['ansible_facts']['distribution_version'] }}
    register: result

  - name: Debug registered variable
    debug:
      var: result.stdout
~~~

ただ、この章は`for`文を使うハンズオンですので無理やりでも使用します。

それでは `Playbook` を実行してみましょう。

~~~console
# ansible-playbook -i inventory/inventory.ini playbooks/playbook.yml
~~~

~~~shell-session
PLAY [docker] ******************************************************************************

TASK [Gathering Facts] *********************************************************************
ok: [dc03]
ok: [dc04]

TASK [Get linux distribution] **************************************************************
changed: [dc03]
changed: [dc04]

TASK [Debug registered variable] ***********************************************************
ok: [dc04] => {
    "result.stdout": "dc04 CentOS 7.7"
}
ok: [dc03] => {
    "result.stdout": "dc03 Alpine 3.11.3"
}

PLAY RECAP *********************************************************************************
dc03                       : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
dc04                       : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
~~~
