# Overview

[Ansible](https://www.ansible.com/) Playground on Docker.

## Launch

Launch Ansible control container and target container.

```bash
$ docker-compose up -d
```

### Run Example Playbook

Run Ansible playbook in the running container.

```bash
$ docker exec -it ansible-playground-control ansible-playbook -i /playbook/ansible-inventory /playbook/shell_touch.yml
# Check generated file.
$ docker exec -it ansible-playground-target cat /tmp/foo
```

Run Playbook in shell.

```bash
$ docker exec -it  ansible-playground-control sh
$ ansible-playbook -i /playbook/ansible-inventory /playbook/shell_touch.yml
```
