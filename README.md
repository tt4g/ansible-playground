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

Ansible 公式のベストプラクティスをベースにしたもの。

See: https://docs.ansible.com/ansible/2.9/user_guide/playbooks_best_practices.html

### プロジェクト構成

Ansible ベストプラクティスで紹介されている以下のプロジェクト構成を踏襲する。

https://docs.ansible.com/ansible/2.9/user_guide/playbooks_best_practices.html#alternative-directory-layout

> ```
> inventories/
>    production/
>       hosts               # inventory file for production servers
>       group_vars/
>          group1.yml       # here we assign variables to particular groups
>          group2.yml
>       host_vars/
>          hostname1.yml    # here we assign variables to particular systems
>          hostname2.yml
>
>    staging/
>       hosts               # inventory file for staging environment
>       group_vars/
>          group1.yml       # here we assign variables to particular groups
>          group2.yml
>       host_vars/
>          stagehost1.yml   # here we assign variables to particular systems
>          stagehost2.yml
>
> library/
> module_utils/
> filter_plugins/
>
> site.yml
> webservers.yml
> dbservers.yml
>
> roles/
>     common/
>     webtier/
>     monitoring/
>     fooapp/
> ```

#### site.yml

`site.yml` をシステムを構成するための起点となる Playbook とする。

**TIP:** `site.yml` 以外の名称を使用しても良いが、システムを構成するための Playbook
ファイルがどれなのかは、分かりやすいように `README.md` に記載しておくこと。

`site.yml` は `import_playbook` で各グループを構成するための Playbook をインポートする。

`site.yml` の例:

```yaml
---
- import_playbook: webservers.yml
- import_playbook: dbservers.yml
```

`site.yml` でインポートされているファイルは、実際にホストを構成するための `role` 定義などを行う。

`webservers.yml` の例:

```yaml
---
- hosts: webservers
  roles:
    - common
    - webtier
```

**TIP:** この構成にすることで、 `site.yml` を実行することでシステム全体を更新する。
`webservers.yml` を実行することで `webservers` グループだけを更新するといった
Ansible の作業対象の分解ができる。

例えば、「システム全体をローリングで再起動する」ようなタスクを実行するための Playbook を
作成した場合も `site.yml` と同じ考え方で Playbook ファイルを作成することを推奨する。

各 Playbook ファイルの役割は、後から分かりやすいように `README.md` に記載しておくこと。

#### inventories

`inventories/` の直下に本番環境、ステージング環境ごとのサブフォルダを用意し、
`hosts` (or `hosts.ini`, `hosts.yml`) に環境ごとの管理ホストを列挙する。

**TIP:** `inventories/<env_name>` の `<env_name>` 部分は環境名であり、
本番環境は `production` のような情報を `README.md` に残しておくこと。

Ansible のベストプラクティスとの相違点として `inventories/<env_name>/group_vars/` の
直下には `hosts` ファイルに列挙されているグループ名のディレクトリを用意し、
その配下に `group_vars/<group_name>/<role_name>.yml` の形式で各 `role` の変数を定義した
ファイルを設置する。
この構成を守ることで、 `role` の変数がどこで定義されているか探しやすくなる。

Ansible のコアモジュール変数は `group_vars/<group_name>/ansible.yml` に定義する。

全てのグループで共通の変数は `group_vars/all` に定義する。
Ansible のドキュメントに記載されているが、全てのホストは暗黙のルールで `all` グループに
属しているため、 `group_vars/all` で定義された変数は全てのホストに自動的に適用される。

`inventories/<env_name>/host_vars` も原則として `inventories/<env_name>/group_vars`
のルールを踏襲する。

`intro_inventory/` 以下の各ファイルに記載する内容については
https://docs.ansible.com/ansible/2.9/user_guide/intro_inventory.html
を参照する。

#### roles

`roles/` 以下の構成は
https://docs.ansible.com/ansible/2.9/user_guide/playbooks_reuse_roles.html
を参照。

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
