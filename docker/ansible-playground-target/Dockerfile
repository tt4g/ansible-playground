FROM python:3.9.6-buster

ARG ROOT_PASSEORD=foobarbaz

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
    openssh-server \
    && apt-get -y clean \
    && rm -rf /var/lib/apt/lists/*

# Enable root login via ssh.
RUN echo "root:${ROOT_PASSEORD}" | chpasswd \
    && mkdir /var/run/sshd \
    && sed -ri 's/^#?PermitRootLogin\s+.*/PermitRootLogin yes/' /etc/ssh/sshd_config \
    # SSH login fix. Otherwise user is kicked off after login
    && sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd \
    && service ssh start

#EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
