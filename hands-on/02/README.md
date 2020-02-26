# 02.インベントリ変数

## 実行環境

~~~console
# pipenv shell
# cd hands-on/02
~~~

## インベントリファイル
インベントリファイルで定義した変数を別ファイルに分離します。分離するにはAnsibleの命名規則にそったディレクトリ名で作成する必要があります。（`group_vars`と`host_vars`）
~~~console
# mkdir -p inventory/group_vars/all
# vi inventory/group_vars/all/login.yml
~~~
~~~yml
---
ansible_ssh_user: root
ansible_ssh_port: 10022
~~~
~~~console
# vi inventory/group_vars/all/python.yml
~~~
~~~yml
---
ansible_python_interpreter: /usr/local/bin/python
~~~
~~~console
# vi inventory/inventory.ini
~~~
~~~ini
[docker]
localhost
~~~

[01.入門](../02/README.md)と同じ内容ではあるものの、インベントリ変数を別ファイルに分離した上で、以下を実行することができる。
~~~console
# ansible-playbook -i inventory/inventory.ini playbooks/playbook.yml
~~~
