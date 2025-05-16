FROM ubuntu:24.04

RUN apt-get update && apt-get install -y \
    software-properties-common \
    build-essential \
    curl \
    wget \
    gnupg2 \
    apt-transport-https \
    ca-certificates \
    openssh-server && \
    mkdir /var/run/sshd && \
    echo 'root:root' | chpasswd && \
    sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config

EXPOSE 22

CMD service ssh start && tail -f /dev/null