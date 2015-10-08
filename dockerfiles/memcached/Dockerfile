FROM ubuntu

RUN apt-get update
RUN \
  apt-get install -y memcached && \
  apt-get clean

EXPOSE 11211

CMD ["/usr/bin/memcached", "-vv"]
USER memcache
