# Overview

[Ansible](https://www.ansible.com/) Playground on Docker.

## Requirements

* Ansible 2.9 (RedHat latest release version)

## Launch

Ansible がインストールされたコンテナと操作対象のコンテナを起動する。

```bash
$ docker-compose up -d
```

### Run Playbook

Ansible の Playbook [ansible/site.yml](./ansible/site.yml) を実行する。

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
