# 11.blockディレクティブ

## 実行環境

~~~console
# pipenv shell
# cd hands-on/11
~~~

## インベントリ

[10.loopディレクティブ](../10/README.md) のインベントリをそのまま使用します。

## プレイブックの編集

`block` ディレクティブを使用したプレイブックの内容ですが、`dc03`/`dc04` に `nginx` を立ち上げて `dc05` から `HAProxy` を経由して `dc03` or `dc04` に投げる構成を作成することにします。

ではプレイブックを作成してみましょう。

~~~console
# vi playbooks/playbook.yml
~~~

~~~yaml
---
- hosts: docker
  connection: docker
  tasks:

  # NGINXを起動
  - block:
    # NGINXをインストール(Alpine)
    - block:
      - name: install openrc for alpine
        apk:
          name: openrc
          state: present

      - name: mkdir openrc running directory for alpine
        file:
          path: /run/openrc
          state: directory
          owner: root
          group: root
          mode: 0755

      - name: touch openrc softlevel file for alpine
        file:
          path: /run/openrc/softlevel
          state: touch
          owner: root
          group: root
          mode: 0755

      - name: start openrc for alpine
        shell: openrc

      - name: install nginx for alpine
        apk:
          name: nginx
          state: present

      - name: mkdir nginx pid directory for alpine
        file:
          path: /run/nginx
          state: directory
          owner: nginx
          group: nginx
          mode: 0755

      - name: start nginx for alpine
        service:
          name: nginx
          enabled: yes
          state: started

      when: ansible_os_family == "Alpine"

    # NGINXをインストール(RedHat)
    - block:
      - name: copy nginx repository for redhat
        copy:
          src: nginx/nginx.repo
          dest: /etc/yum.repos.d/
          owner: root
          group: root
          mode: 0644

      - name: install nginx for redhat
        yum:
          name: nginx
          state: present

      - name: start nginx for redhat
        systemd:
          name: nginx
          enabled: yes
          state: started

      when: ansible_os_family == "RedHat"

    - name: mkdir nginx root directory
      file:
        path: /var/www/html
        state: directory
        owner: root
        group: root
        mode: 0755

    - name: copy nginx configuration
      copy:
        src: nginx/nginx.conf
        dest: /etc/nginx/conf.d/default.conf
        owner: root
        group: root
        mode: 0644

    - name: copy nginx configuration
      template:
        src: contents/top.html.j2
        dest: /var/www/html/top.html
        owner: root
        group: root
        mode: 0644

    - name: reload nginx for alpine
      shell: nginx -s reload
      when: ansible_os_family == "Alpine"

    - name: restart nginx for redhat
      systemd:
        name: nginx
        enabled: yes
        state: restarted
      when: ansible_os_family == "RedHat"

    when: |
      ( ansible_hostname == "dc03" ) or
      ( ansible_hostname == "dc04" )

  # HAProxyを起動
  - block:
    - name: install haproxy for debian
      apt:
        name: haproxy
        state: present
      when: ansible_os_family == "Debian"

    - name: start haproxy for debian
      service:
        name: haproxy
        state: started
      when: ansible_os_family == "Debian"

    - name: copy haproxy configuration
      copy:
        src: haproxy/haproxy.cfg
        dest: /etc/haproxy/haproxy.cfg
        owner: root
        group: root
        mode: 0644

    - name: restart haproxy for debian
      service:
        name: haproxy
        state: restarted
      when: ansible_os_family == "Debian"

    when: ansible_hostname == "dc05"
~~~

それでは `Playbook` を実行してみましょう。

~~~console
# ansible-playbook -i inventory/inventory.ini playbooks/playbook.yml
~~~

~~~shell-session
PLAY [docker] ******************************************************************************

TASK [Gathering Facts] *********************************************************************
ok: [dc03]
ok: [dc04]
ok: [dc05]

TASK [install openrc for alpine] ***********************************************************
skipping: [dc04]
skipping: [dc05]
changed: [dc03]

TASK [mkdir openrc running directory for alpine] *******************************************
skipping: [dc04]
skipping: [dc05]
changed: [dc03]

TASK [touch openrc softlevel file for alpine] **********************************************
skipping: [dc04]
skipping: [dc05]
changed: [dc03]

TASK [install nginx for alpine] ************************************************************
skipping: [dc04]
skipping: [dc05]
changed: [dc03]

TASK [mkdir nginx pid directory for alpine] ************************************************
skipping: [dc04]
skipping: [dc05]
changed: [dc03]

TASK [start openrc for alpine] *************************************************************
skipping: [dc04]
skipping: [dc05]
changed: [dc03]

TASK [start nginx for alpine] **************************************************************
skipping: [dc04]
skipping: [dc05]
changed: [dc03]

TASK [copy nginx repository for redhat] ****************************************************
skipping: [dc03]
skipping: [dc05]
changed: [dc04]

TASK [install nginx for redhat] ************************************************************
skipping: [dc03]
skipping: [dc05]
changed: [dc04]

TASK [start nginx for redhat] **************************************************************
skipping: [dc03]
skipping: [dc05]
changed: [dc04]

TASK [mkdir nginx root directory] **********************************************************
skipping: [dc05]
changed: [dc04]
changed: [dc03]

TASK [copy nginx configuration] ************************************************************
skipping: [dc05]
changed: [dc04]
changed: [dc03]

TASK [copy nginx configuration] ************************************************************
skipping: [dc05]
changed: [dc04]
changed: [dc03]

TASK [reload nginx for alpine] *************************************************************
skipping: [dc04]
skipping: [dc05]
changed: [dc03]

TASK [restart nginx for redhat] ************************************************************
skipping: [dc03]
skipping: [dc05]
changed: [dc04]

TASK [install haproxy for debian] **********************************************************
skipping: [dc03]
skipping: [dc04]
[WARNING]: Updating cache and auto-installing missing dependency: python-apt
changed: [dc05]

TASK [start haproxy for debian] ************************************************************
skipping: [dc03]
skipping: [dc04]
changed: [dc05]

TASK [copy haproxy configuration] **********************************************************
skipping: [dc03]
skipping: [dc04]
changed: [dc05]

TASK [restart haproxy for debian] **********************************************************
skipping: [dc03]
skipping: [dc04]
changed: [dc05]

PLAY RECAP *********************************************************************************
dc03                       : ok=12   changed=11   unreachable=0    failed=0    skipped=8    rescued=0    ignored=0
dc04                       : ok=8    changed=7    unreachable=0    failed=0    skipped=12   rescued=0    ignored=0
dc05                       : ok=5    changed=4    unreachable=0    failed=0    skipped=15   rescued=0    ignored=0
~~~

実際に `dc05` から `dc03` or `dc04` の `nginx` へアクセスしてみましょう。

~~~console
# docker exec -it dc05 bash
root@dc05:/# curl http://localhost/top.html
~~~

~~~html
<html xmlns="http://www.w3.org/1999/xhtml" lang="ja">
 <head>
  <title>ホスト名は「dc03」です</title>
 </head>
</html>
~~~

もう一度 `dc03` or `dc04` の `nginx` へアクセスしてみましょう。

~~~console
root@dc05:/# curl http://localhost/top.html
~~~

~~~html
<html xmlns="http://www.w3.org/1999/xhtml" lang="ja">
 <head>
  <title>ホスト名は「dc04」です</title>
 </head>
</html>
~~~

`HAProxy` が動作していることを確認できました。
