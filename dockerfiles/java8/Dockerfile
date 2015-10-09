FROM ubuntu

RUN \
  apt-get update && \
  apt-get install -y \
    software-properties-common \
    unzip \
    curl

RUN \
  echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | debconf-set-selections && \
  add-apt-repository -y ppa:webupd8team/java && \
  apt-get update && \
  apt-get install -y oracle-java8-installer && \
  apt-get clean && \
  rm -rf /var/cache/oracle-jdk8-installer

RUN \
  groupadd -r build && \
  useradd -m -g build mvn

RUN \
  curl -L \
    http://ftp.yz.yamagata-u.ac.jp/pub/network/apache/maven/maven-3/3.3.3/binaries/apache-maven-3.3.3-bin.zip \
    -o /tmp/maven3.zip

RUN \
  unzip -q tmp/maven3.zip -d tmp/maven && \
  mv tmp/maven/apache-maven-3.3.3/ opt/maven && \
  rm -rf tmp/*

USER mvn
WORKDIR /home/mvn

ENV JAVA_HOME /usr/lib/jvm/java-8-oracle
ENV PATH=/opt/maven/bin:$PATH
EXPOSE 8080

CMD ["/bin/bash"]
