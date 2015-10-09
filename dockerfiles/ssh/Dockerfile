FROM ubuntu

RUN \
  apt-get update && \
  apt-get install -y openssh-server && \
  apt-get clean && \
  mkdir -p /var/run/sshd

RUN \
  useradd -m mike && \
  mkdir -p /home/mike/.ssh && \
  chown mike /home/mike/.ssh && \
  chmod 700 /home/mike/.ssh

ADD ./authorized_keys /home/mike/.ssh/authorized_keys

RUN \
  chown mike /home/mike/.ssh/authorized_keys && \
  chmod 600 /home/mike/.ssh/authorized_keys

CMD ["/usr/sbin/sshd","-D"]
