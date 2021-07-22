FROM python:3.9.6-buster

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
    # Install sshpass for using Ansible ssh.
    sshpass \
    && apt-get -y clean \
    && rm -rf /var/lib/apt/lists/*

RUN python -m pip install --upgrade pip==21.1.3 \
    && python -m pip install ansible==4.3.0
