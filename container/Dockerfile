FROM python:3-alpine

RUN set -x && apk update && apk upgrade \
  && apk add --no-cache openssh openrc bash

RUN rc-update add sshd && rc-status
RUN mkdir -p /run/openrc/ && touch /run/openrc/softlevel

# パスワードなしでログイン可能にする
RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config

# 公開キーの配置
RUN mkdir /root/.ssh && chmod 700 /root/.ssh
COPY id_rsa.pub /root/.ssh/authorized_keys
RUN chmod 600 /root/.ssh/authorized_keys

# パスワードを設定しないとログインできない
RUN passwd -d root

EXPOSE 22
CMD /etc/init.d/sshd start && tail -f /dev/null
