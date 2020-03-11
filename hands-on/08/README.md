# 08.変数フィルタ

## 実行環境

~~~console
# pipenv shell
# cd hands-on/08
~~~

## インベントリ

[07.定義済み変数とfor文](../07/README.md) のインベントリをそのまま使用します。

## プレイブックの編集

変数フィルタを使用してプレイブックを作成してみましょう。

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
      echo {{ hostvars[ansible_hostname]['ansible_facts']['distribution'] }} {{ hostvars[ansible_hostname]['ansible_facts']['distribution_version'] }}
    register: dist

  - name: ディストリビュージョンを表示
    debug:
      var: dist.stdout

  - name: 大文字に変換
    debug:
      var: dist.stdout | upper

  - name: 小文字に変換
    debug:
      var: dist.stdout | lower

  - name: 文字列数に変換
    debug:
      var: dist.stdout | length

  - name: 変数がない場合はデフォルト値に変換
    debug:
      var: dist.stdoutxx | default('デフォルト')

  - name: 配列の結合、文字列も文字の配列みたいなもの
    debug:
      var: dist.stdout | join(' ')
~~~

いくつかの変数フィルタを使用してみました。ただ、これ以上にもたくさんあります。

ここではやりませんので、詳細は [公式](https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters.html) を参照してください。ここらへんは`Python`ユーザーであれば欲しいフィルタを探すのは難しくないと思います。`Python`のビルドイン関数や標準モジュールの関数名と類似してますので。

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

TASK [ディストリビュージョンを表示] **********************************************************************
ok: [dc03] => {
    "dist.stdout": "Alpine 3.11.3"
}
ok: [dc04] => {
    "dist.stdout": "CentOS 7.7"
}

TASK [大文字に変換] ******************************************************************************
ok: [dc03] => {
    "dist.stdout | upper": "ALPINE 3.11.3"
}
ok: [dc04] => {
    "dist.stdout | upper": "CENTOS 7.7"
}

TASK [小文字に変換] ******************************************************************************
ok: [dc03] => {
    "dist.stdout | lower": "alpine 3.11.3"
}
ok: [dc04] => {
    "dist.stdout | lower": "centos 7.7"
}

TASK [文字列数に変換] *****************************************************************************
ok: [dc03] => {
    "dist.stdout | length": "13"
}
ok: [dc04] => {
    "dist.stdout | length": "10"
}

TASK [変数がない場合はデフォルト値に変換] *******************************************************************
ok: [dc03] => {
    "dist.stdoutxx | default('デフォルト')": "デフォルト"
}
ok: [dc04] => {
    "dist.stdoutxx | default('デフォルト')": "デフォルト"
}

TASK [配列の結合、文字列も文字の配列みたいなもの] ***************************************************************
ok: [dc03] => {
    "dist.stdout | join(' ')": "A l p i n e   3 . 1 1 . 3"
}
ok: [dc04] => {
    "dist.stdout | join(' ')": "C e n t O S   7 . 7"
}

PLAY RECAP *********************************************************************************
dc03                       : ok=8    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
dc04                       : ok=8    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
~~~
