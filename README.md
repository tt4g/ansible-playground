# Overview

[Ansible](https://www.ansible.com/) Playground on Docker.

## Requirements

* Ansible 2.9 (RedHat latest release version)

## Launch

Ansible がインストールされたコンテナと操作対象のコンテナを起動する。

```bash
$ docker-compose up -d
```

## Ansible

### SSH fingerprint error

ホストサーバーにSSH接続をするとき、 fingerprint が登録されていないと Ansible は Playbook
の実行時に次のエラーメッセージを表示して失敗することがある。

```
Using a SSH password instead of a key is not possible because Host Key checking is enabled and sshpass does not support this.  Please add this host's fingerprint to your known_hosts file to manage this host.
```

fingerprint が登録されていないサーバーへの接続が拒否されているため、
次のいずれかの方法で問題を解決する。

* 対象のサーバーに一度 SSH 接続を行い、 fingerprint を登録する
* `export ANSIBLE_HOST_KEY_CHECKING=False` で Ansible の鍵チェックを無効化する
  もしくは `ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook ...` でコマンドを実行する

一度 fingerprint が登録されれば、このエラーは発生しなくなる。

ホストに確実に接続できると判断できる状況で、多くの新規ホストで Playbook を実行する場合には
`ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook ...` で fingerprint を
まとめて登録しながら Playbook を実行することが出来る。

#### 構成変更

[ansible/site.yml](./ansible/site.yml) を実行する。

```bash
$ docker-compose exec -it ansible-playground-control ansible-playbook \
  -i /ansible/inventories/development/hosts.ini \
  /ansible/site.yml
# Check generated file.
$ docker-compose exec ansible-playground-debian-bullseye cat /tmp/ansible_hostname
$ docker-compose exec ansible-playground-redhat-8 cat /tmp/ansible_hostname
```

コンテナ内にアクセスして実行する。

```bash
$ docker-compose exec ansible-playground-control /bin/bash -l
$ ansible-playbook \
  -i /ansible/inventories/development/hosts.ini \
  /ansible/site.yml
```

### 構成の検証

[ansible/site.yml](./ansible/site.yml) を実行する。

```bash
$ docker-compose exec -it ansible-playground-control ansible-playbook \
  -i /ansible/inventories/development/hosts.ini \
  /ansible/assert_system_configuration.yml
```
