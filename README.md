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

### Playbook

#### 構成変更

[ansible/site.yml](./ansible/site.yml) を実行する。

```bash
$ docker-compose exec -it ansible-playground-control ansible-playbook \
  -i /ansible/inventories/development/hosts.ini \
  --vault-password-file /ansible/.vault_password \
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
  --vault-password-file /ansible/.vault_password \
  /ansible/site.yml
```

#### 構成の検証

[ansible/site.yml](./ansible/site.yml) を実行する。

```bash
$ docker-compose exec -it ansible-playground-control ansible-playbook \
  -i /ansible/inventories/development/hosts.ini \
  --vault-password-file /ansible/.vault_password \
  /ansible/assert_system_configuration.yml
```

## ベストプラクティス

### 機密情報の暗号化

Ansible でパスワードなどの機密情報を暗号化して保存するために
Ansible Vault (`ansible-vault`) を使用する。

`ansible-vault encrypt path/to/file` でファイルの内容を暗号化できるが、ファイル内の
**変数名も暗号化されてしまう**。
変数名が暗号化されると該当ファイル内で定義されている変数が分からなくなってしまう。

Ansible のベストプラクティス
(https://docs.ansible.com/ansible/2.9/user_guide/playbooks_best_practices.html#variables-and-vaults)
にのっとり、ここでは次の手順で `ansible-vault` による暗号化を行う。

暗号化元ファイル `group_vars/foo_group/vars.yml` を作成し、暗号化対象の変数を定義する。
該当の変数は Jinja2 シンタックスで `vault__` プレフィクスを付与した変数を参照する。

**TIP:** こうしておくことで `grep` などで変数の定義元を探しやすくなる。

```yaml
secure_text: "{{ vault__secure_text }}"
```

暗号化するファイル `group_vars/foo_group/vars__vault.yml` を作成し、暗号化対象の変数を定義する。

```yaml
vault__secure_text: "!!!login password!!!"
```

`group_vars/foo_group/vars__vault.yml` を `ansible-vault encrypt` コマンドで暗号化する。

```bash
$ ansible-vault encrypt group_vars/foo_group/vars__vault.yml
New Vault password: 
Confirm New Vault password: 
Encryption successful
```

暗号化されたファイルは内容が全て暗号化されるため、`git` などのリポジトリで管理する。
ただし、`ansible-vault` で暗号化に使用したパスワードは `git` などのリポジトリとは別に管理する事。

**WARNING:** このリポジトリは Playground 用のリポジトリなので
 [ansible/.vault_password](./ansible/.vault_password) で暗号化パスワードを管理しているが、
 実際のプロジェクトでは決して暗号化済みファイルと複合用パスワードを同じ場所で管理してはいけない。

Ansible Vault で暗号化されたファイルを読み込むときは、 `ansible-playbook` コマンドに
引数 `--ask-vault-pass` を追加する。
パスワードをファイルに保存している場合は
`--vault-password-file /path/to/.vault_password` を追加する。

Example:

* `ansible-playbook -i inventories/development/hosts.ini --ask-vault-pass site.yml`
* `ansible-playbook -i inventories/development/hosts.ini --vault-password-file .vault_password site.yml`


暗号化されたファイルの内容は `ansible-vault view /path/to/encrypted` で表示できる。
差分更新は `ansible-vault edit /path/to/encrypted` で可能となっており、
複合は `ansible-vault decript /path/to/encrypted` で出来る。
