# https://hub.docker.com/r/redhat/ubi8-init
FROM redhat/ubi8-init:8.5-10.1645811924

ARG USER_UID=1000
ARG USER_GID=1000
ARG USER_USERNAME=worker
ARG USER_GROUPNAME=worker
ARG USER_PASSWORD=worker

RUN dnf -y install \
    sudo openssh-server \
    && dnf clean all

# SSH login fix. Otherwise user is kicked off after login
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd \
    && systemctl enable sshd

# Add user and group.
RUN groupadd -g ${USER_GID} ${USER_GROUPNAME} \
    && useradd -m -s /bin/bash -u ${USER_UID} -g ${USER_GID} ${USER_USERNAME} \
    && echo "${USER_USERNAME}:${USER_PASSWORD}" | chpasswd \
    && echo "${USER_USERNAME}   ALL=(ALL:ALL) ALL" > "/etc/sudoers.d/${USER_USERNAME}" \
    && chmod 0440 "/etc/sudoers.d/${USER_USERNAME}"
