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
~~~

それでは `Playbook` を実行してみましょう。

~~~console
# ansible-playbook -i inventory/inventory.ini playbooks/playbook.yml
~~~

### TODO：まだ書きかけなの

~~~shell-session
~~~

実際に `dc05` から `dc03` or `dc04` の `nginx` へアクセスしてみましょう。

~~~console
# docker exec -it dc05 bash
~~~

~~~console
root@dc05:/# curl http://localhost:8080
~~~
