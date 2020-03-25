# 12.rolesディレクティブ

## 実行環境

~~~console
# pipenv shell
# cd hands-on/12
~~~

## インベントリの編集

[11.blockディレクティブ](../11/README.md) のインベントリを少し編集して使用します。

~~~console
# vi inventory/inventory.ini
~~~

<details><summary>inventory/inventory.ini</summary><p>

~~~ini
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

[web]
dc03
dc04

[proxy]
dc05
~~~

</p></details>

新たに `web` と `proxy` グループを指定することにしました。

ここで分けなくともプレイブックの `hosts` で複数のホストを指定すればいいだけなんですけどね。

...ぅ、指定した方がわかりやすいかなって思っただけです。

## プレイブックの編集

内容は [11.blockディレクティブ](../11/README.md) のプレイブックと同様に `dc03`/`dc04` に `nginx` を立ち上げて `dc05` から `HAProxy` を経由して `dc03` or `dc04` に投げる構成です。

[11.blockディレクティブ](../11/README.md) のプレイブックが読みづらいものでしたが、`roles`ディレクティブを使用することでシンプルにできます。

~~~console
# vi playbooks/playbook.yml
~~~

<details><summary>playbooks/playbook.yml</summary><p>

~~~yaml
---
- hosts: web
  connection: docker
  roles:
    - nginx

- hosts: proxy
  connection: docker
  roles:
    - haproxy
~~~

</p></details>

**読みやすい！** **シンプル！**

ただですね、これだけではもちろん実行できませんので、実行可能にするためにロールを作成していきます。

プレイブック中の `roles` ディレクティブで指定している `nginx` と `haproxy` がロールです。

ロールを作成するディレクトリは `roles` というディレクトリ名にすると決まっています。

~~~console
# mkdir playbooks/roles
~~~

`roles` ディレクトリ以下のディレクトリ名に関しても名前が決まっていますので注意してください。

### nginx ロールの作成

それでは `nginx` のロールを作成してみましょう。ディレクトリ名がロール名となります。

~~~console
# mkdir playbooks/roles/nginx
~~~

必要となる設定ファイルなどは `files` ディレクトリに配置します。

~~~
# mkdir playbooks/roles/nginx/files
# vi playbooks/roles/nginx/files/nginx.conf
~~~

<details><summary>playbooks/roles/nginx/files/nginx.conf</summary><p>

~~~nginx
# This is a default site configuration which will simply return 404, preventing
# chance access to any other virtualhost.

server {
        listen 80 default_server;
        listen [::]:80 default_server;

        # change root directory
        location / {
                root /var/www/html;
        }

        # You may need this to prevent return 404 recursion.
        location = /404.html {
                internal;
        }
}
~~~

</p></details>

~~~console
# vi playbooks/roles/nginx/files/nginx.repo
~~~

<details><summary>playbooks/roles/nginx/files/nginx.repo</summary><p>

~~~ini
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1
~~~

</p></details>

`templete` モジュールで使用するファイルは `templates` ディレクトリへ配置します。

~~~console
# mkdir playbooks/roles/nginx/templates
# vi playbooks/roles/nginx/templates/top.html.j2
~~~

<details><summary>playbooks/roles/nginx/templates/top.html.j2</summary><p>

~~~jinja
<html xmlns="http://www.w3.org/1999/xhtml" lang="ja">
 <head>
  <title>ホスト名は「{{ ansible_hostname }}」です</title>
 </head>
</html>
~~~

</p></details>

必要なファイルを配置したところでロールの`YAML`ファイルを作成します。
こちらは `tasks` ディレクトリに `main.yml` という名前で作成する必要があります。

~~~console
# mkdir playbooks/roles/nginx/tasks
# vi playbooks/roles/nginx/tasks/main.yml
~~~

<details><summary>playbooks/roles/nginx/tasks/main.yml</summary><p>

~~~yaml
---
# NGINXのインストール(Alpine)
- include_tasks: install_nginx_for_alpine.yml
  tags: install_nginx_for_alpine
  when: ansible_os_family == "Alpine"

# NGINXのインストール(RedHat)
- include_tasks: install_nginx_for_redhat.yml
  tags: install_nginx_for_redhat
  when: ansible_os_family == "RedHat"
~~~

</p></details>

上記では `install_nginx_for_alpine.yml` と `install_nginx_for_redhat.yml` を `include` しています。

~~~console
# vi playbooks/roles/nginx/tasks/install_nginx_for_alpine.yml
~~~

<details><summary>playbooks/roles/nginx/tasks/install_nginx_for_alpine.yml</summary><p>

~~~yaml
---
# nginx をサービスとして起動するために必要な OpenRC をインストール
- include_tasks: install_openrc_for_alpine.yml

# nginx のインストール
- name: install nginx for Alpine
  apk:
    name: nginx
    state: present

# PIDファイルが作成されるディレクトリを作成
- name: mkdir nginx pid directory for Alpine
  file:
    path: /run/nginx
    state: directory
    owner: nginx
    group: nginx
    mode: 0755

# nginx を起動
- name: start nginx for Alpine
  service:
    name: nginx
    enabled: yes
    state: started

# nginx の設定変更を適用
- include_tasks: setup_nginx.yml
~~~

</p></details>

コンテナのOSの一つに `Alpine` を選択した（してしまった）ので `nginx` をサービスとして起動するために `OpenRC` のインストールが必要となります。

余談ですが、コンテナのOSを `centos` に統一しておけばよかったと後から思いました。最初は複数のOSでやった方が勉強になるかな、と思ったのですが結果的に `ansible` 以外のところで時間を取られてしまいました。

`Alpine` だけでなく `RedHat` 用のファイルも作成します。

~~~console
# vi playbooks/roles/nginx/tasks/install_nginx_for_redhat.yml
~~~

<details><summary>playbooks/roles/nginx/tasks/install_nginx_for_redhat.yml</summary><p>

~~~yaml
---
# nginx のリポジトリを追加
- name: copy nginx repository for Redhat
  copy:
    src: nginx.repo
    dest: /etc/yum.repos.d/
    owner: root
    group: root
    mode: 0644

# nginx のインストール
- name: install nginx for Redhat
  yum:
    name: nginx
    state: present

# nginx を起動
- name: start nginx for Redhat
  systemd:
    name: nginx
    enabled: yes
    state: started

# nginx の設定変更を適用
- include_tasks: setup_nginx.yml
~~~

</p></details>

先にのべたように `Alpine` の場合は `OpenRC` のインストールも必要となります。

~~~console
# vi playbooks/roles/nginx/tasks/install_openrc_for_alpine.yml
~~~

<details><summary>playbooks/roles/nginx/tasks/install_openrc_for_alpine.yml</summary><p>

~~~yaml
---
# OpenRC のインストール
- name: install openrc for Alpine
  apk:
    name: openrc
    state: present

# PIDファイル用のディレクトリを作成
- name: mkdir openrc running directory for Alpine
  file:
    path: /run/openrc
    state: directory
    owner: root
    group: root
    mode: 0755

# 起動回避
- name: touch openrc softlevel file for Alpine
  file:
    path: /run/openrc/softlevel
    state: touch
    owner: root
    group: root
    mode: 0755

# 起動
- name: start openrc for Alpine
  shell: openrc
~~~

</p></details>

また、`nginx` の設定ファイルの設定として `install_nginx_for_alpine.yml` と `install_nginx_for_redhat.yml` の中で `setup_nginx.yml` を `include` しています。

~~~console
# vi playbooks/roles/nginx/tasks/setup_nginx.yml
~~~

<details><summary>playbooks/roles/nginx/tasks/setup_nginx.yml</summary><p>

~~~yaml
---
# ルートディレクトリを作成
- name: mkdir nginx root directory
  file:
    path: /var/www/html
    state: directory
    owner: root
    group: root
    mode: 0755

# nginx の設定ファイルをコピー、設定ファイルに変更があった場合は nginx を再起動
- name: copy nginx configuration
  copy:
    src: nginx.conf
    dest: /etc/nginx/conf.d/default.conf
    owner: root
    group: root
    mode: 0644
  notify: restart nginx for {{ ansible_os_family }}

# htmlファイルを配置
- name: copy nginx configuration
  template:
    src: top.html.j2
    dest: /var/www/html/top.html
    owner: root
    group: root
    mode: 0644
~~~

</p></details>

ここで新たなディレクティブ `notify` が登場しました。`notify` はタスクの結果が `changed` だった場合にのみ、別のタスクを処理する仕組みです。

実際のファイルは `handlers` ディレクトリに `main.yml` という名前で配置する必要があります。

~~~console
# mkdir playbooks/roles/nginx/handlers
# vi playbooks/roles/nginx/handlers/main.yml
~~~

<details><summary>playbooks/roles/nginx/handlers/main.yml</summary><p>

~~~yaml
---
- name: restart nginx for Alpine
  shell: nginx -s reload

- name: restart nginx for RedHat
  systemd:
    name: nginx
    enabled: yes
    state: restarted
~~~

</p></details>

これにより `nginx` の設定ファイルが変更された場合に `nginx` が再起動することができます。

また、設定ファイルが変更されなかった場合は `nginx` の再起動は行われません。

これで `nginx` ロールの作成は完了です。

## haproxy ロールの作成

続いて `haproxy` ロールを作成します。
先の `nginx` と同様にロール名をディレクトリ名とします。

~~~console
# mkdir playbooks/roles/haproxy
~~~

`haproxy` の設定ファイルを配置します。

~~~console
# mkdir playbooks/roles/haproxy/files
# vi playbooks/roles/haproxy/files/haproxy.cfg
~~~

<details><summary>playbooks/roles/haproxy/files/haproxy.cfg</summary><p>

~~~ini
global
        log /dev/log    local0
        log /dev/log    local1 notice
        chroot /var/lib/haproxy
        stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
        stats timeout 30s
        user haproxy
        group haproxy
        daemon

        # Default SSL material locations
        ca-base /etc/ssl/certs
        crt-base /etc/ssl/private

        # Default ciphers to use on SSL-enabled listening sockets.
        # For more information, see ciphers(1SSL). This list is from:
        #  https://hynek.me/articles/hardening-your-web-servers-ssl-ciphers/
        # An alternative list with additional directives can be obtained from
        #  https://mozilla.github.io/server-side-tls/ssl-config-generator/?server=haproxy
        ssl-default-bind-ciphers ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:RSA+AESGCM:RSA+AES:!aNULL:!MD5:!DSS
        ssl-default-bind-options no-sslv3

defaults
        log     global
        mode    http
        option  httplog
        option  dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http

frontend web_proxy
    default_backend web_servers
    bind *:80

backend web_servers
    option redispatch
    retries 3
    server web01 dc03:80 check
    server web02 dc04:80 check
~~~

</p></details>

必要なファイルを配置したところでロールの`YAML`ファイルを作成します。
こちらは `tasks` ディレクトリに `main.yml` という名前で作成する必要があります。（デジャブ）

~~~console
# mkdir playbooks/roles/haproxy/tasks
# vi playbooks/roles/haproxy/tasks/main.yml
~~~

<details><summary>playbooks/roles/haproxy/tasks/main.yml</summary><p>

~~~yaml
---
- include_tasks: install_haproxy_for_debian.yml
  tags: install_haproxy_for_debian
~~~

</p></details>

`main.yml` では `install_haproxy_for_debian.yml` を `include` しています。

~~~console
# vi playbooks/roles/haproxy/tasks/install_haproxy_for_debian.yml
~~~

<details><summary>playbooks/roles/haproxy/tasks/install_haproxy_for_debian.yml</summary><p>

~~~yaml
---
- name: install haproxy for Debian
  apt:
    name: haproxy
    state: present

- name: start haproxy for Debian
  service:
    name: haproxy
    state: started

- include_tasks: setup_haproxy.yml
~~~

</p></details>

上ではインストール後に `haproxy` の設定を行うタスクを `include` しています。

~~~console
# vi playbooks/roles/haproxy/tasks/setup_haproxy.yml
~~~

<details><summary>playbooks/roles/haproxy/tasks/setup_haproxy.yml</summary><p>

~~~yaml
---
- name: copy haproxy configuration
  copy:
    src: haproxy.cfg
    dest: /etc/haproxy/haproxy.cfg
    owner: root
    group: root
    mode: 0644
  notify: restart haproxy for {{ ansible_os_family }}
~~~

</p></details>

またまたでてきました `notify` ディレクティブです。
こちらに指定する値は、ハンドラーの名前ですので注意してください。

`haproxy` では `Debian` だけですが、他のOSのことを考えて名前にOSをいれています。

~~~console
# mkdir playbooks/roles/haproxy/handlers
# vi playbooks/roles/haproxy/handlers/main.yml
~~~

<details><summary>playbooks/roles/haproxy/handlers/main.yml</summary><p>

~~~yaml
---
- name: restart haproxy for Debian
  service:
    name: haproxy
    state: restarted
~~~

</p></details>

これで `haproxy` ロールの作成は完了です。

### 実行

それでは `Playbook` を実行してみましょう。

~~~console
# ansible-playbook -i inventory/inventory.ini playbooks/playbook.yml
~~~

<details><summary>実行結果</summary><p>

~~~shell-session
PLAY [web] *********************************************************************************

TASK [Gathering Facts] *********************************************************************
ok: [dc03]
ok: [dc04]

TASK [nginx : include_tasks] ***************************************************************
skipping: [dc04]
included: /.../study.ansible/hands-on/12/playbooks/roles/nginx/tasks/install_nginx_for_alpine.yml for dc03

TASK [nginx : include_tasks] ***************************************************************
included: /.../study.ansible/hands-on/12/playbooks/roles/nginx/tasks/install_openrc_for_alpine.yml for dc03

TASK [nginx : install openrc for Alpine] ***************************************************
changed: [dc03]

TASK [nginx : mkdir openrc running directory for Alpine] ***********************************
changed: [dc03]

TASK [nginx : touch openrc softlevel file for Alpine] **************************************
changed: [dc03]

TASK [nginx : start openrc for Alpine] *****************************************************
changed: [dc03]

TASK [nginx : install nginx for Alpine] ****************************************************
changed: [dc03]

TASK [nginx : mkdir nginx pid directory for Alpine] ****************************************
changed: [dc03]

TASK [nginx : start nginx for Alpine] ******************************************************
changed: [dc03]

TASK [nginx : include_tasks] ***************************************************************
included: /.../study.ansible/hands-on/12/playbooks/roles/nginx/tasks/setup_nginx.yml for dc03

TASK [nginx : mkdir nginx root directory] **************************************************
changed: [dc03]

TASK [nginx : copy nginx configuration] ****************************************************
changed: [dc03]

TASK [nginx : copy nginx configuration] ****************************************************
changed: [dc03]

TASK [nginx : include_tasks] ***************************************************************
skipping: [dc03]
included: /.../study.ansible/hands-on/12/playbooks/roles/nginx/tasks/install_nginx_for_redhat.yml for dc04

TASK [nginx : copy nginx repository for Redhat] ********************************************
changed: [dc04]

TASK [nginx : install nginx for Redhat] ****************************************************
changed: [dc04]

TASK [nginx : start nginx for Redhat] ******************************************************
changed: [dc04]

TASK [nginx : include_tasks] ***************************************************************
included: /.../study.ansible/hands-on/12/playbooks/roles/nginx/tasks/setup_nginx.yml for dc04

TASK [nginx : mkdir nginx root directory] **************************************************
changed: [dc04]

TASK [nginx : copy nginx configuration] ****************************************************
changed: [dc04]

TASK [nginx : copy nginx configuration] ****************************************************
changed: [dc04]

RUNNING HANDLER [nginx : restart nginx for Alpine] *****************************************
changed: [dc03]

RUNNING HANDLER [nginx : restart nginx for RedHat] *****************************************
changed: [dc04]

PLAY [proxy] *******************************************************************************

TASK [Gathering Facts] *********************************************************************
ok: [dc05]

TASK [haproxy : include_tasks] *************************************************************
included: /.../study.ansible/hands-on/12/playbooks/roles/haproxy/tasks/install_haproxy_for_debian.yml for dc05

TASK [haproxy : install haproxy for Debian] ************************************************
[WARNING]: Updating cache and auto-installing missing dependency: python-apt
changed: [dc05]

TASK [haproxy : start haproxy for Debian] **************************************************
changed: [dc05]

TASK [haproxy : include_tasks] *************************************************************
included: /.../study.ansible/hands-on/12/playbooks/roles/haproxy/tasks/setup_haproxy.yml for dc05

TASK [haproxy : copy haproxy configuration] ************************************************
changed: [dc05]

RUNNING HANDLER [haproxy : restart haproxy for Debian] *************************************
changed: [dc05]

PLAY RECAP *********************************************************************************
dc03                       : ok=15   changed=11   unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
dc04                       : ok=10   changed=7    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
dc05                       : ok=7    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
~~~

</p></details>

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

[11.blockディレクティブ](../11/README.md)と同様の結果が得られました。

結果は同じでも `playbooks` 以下の構成は別物になっています。

ロールを作成するのは大変ですが、一度作成すれば遺産として残りますし、プレイブックがシンプルになりますので、サーバー構築が簡単になるのではないでしょうか。
