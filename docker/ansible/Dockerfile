ARG PYTHON_VERSION=3.10.2

FROM python:${PYTHON_VERSION}-bullseye

ARG ANSIBLE_UID=1000
ARG ANSIBLE_GID=1000
ARG ANSIBLE_USERNAME=ansible
ARG ANSIBLE_GROUPNAME=ansible
ARG ANSIBLE_PASSWORD=ansible
ARG ANSIBLE_VERSION=2.9.18

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
    # Install sshpass for using Ansible ssh.
    sshpass \
    git sudo \
    && apt-get -y clean \
    && rm -rf /var/lib/apt/lists/*

RUN python -m pip install --upgrade pip==22.0.3 \
    && python -m pip install ansible==${ANSIBLE_VERSION} ansible-lint==5.4.0

# Add user and group.
RUN groupadd -g ${ANSIBLE_GID} ${ANSIBLE_GROUPNAME} \
    && useradd -m -s /bin/bash -u ${ANSIBLE_UID} -g ${ANSIBLE_GID} ${ANSIBLE_USERNAME} \
    && echo "${ANSIBLE_USERNAME}:${ANSIBLE_PASSWORD}" | chpasswd \
    && echo "${ANSIBLE_USERNAME}   ALL=(ALL:ALL) ALL" > "/etc/sudoers.d/${ANSIBLE_GROUPNAME}" \
    && chmod 0440 "/etc/sudoers.d/${ANSIBLE_GROUPNAME}"

USER ${ANSIBLE_UID}:${ANSIBLE_GID}
