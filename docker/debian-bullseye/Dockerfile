ARG PYTHON_VERSION=3.10.2

FROM python:${PYTHON_VERSION}-bullseye

ARG USER_UID=1000
ARG USER_GID=1000
ARG USER_USERNAME=worker
ARG USER_GROUPNAME=worker
ARG USER_PASSWORD=worker

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
    openssh-server sudo \
    && apt-get -y clean \
    && rm -rf /var/lib/apt/lists/*

# Enable ssh server.
RUN mkdir /var/run/sshd \
    # SSH login fix. Otherwise user is kicked off after login
    && sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd

# Add user and group.
RUN groupadd -g ${USER_GID} ${USER_GROUPNAME} \
    && useradd -m -s /bin/bash -u ${USER_UID} -g ${USER_GID} ${USER_USERNAME} \
    && echo "${USER_USERNAME}:${USER_PASSWORD}" | chpasswd \
    && echo "${USER_USERNAME}   ALL=(ALL:ALL) ALL" > "/etc/sudoers.d/${USER_USERNAME}" \
    && chmod 0440 "/etc/sudoers.d/${USER_USERNAME}"

CMD ["/usr/sbin/sshd", "-D"]
