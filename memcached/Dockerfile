FROM heremedia/acquia-base
MAINTAINER Nicholas Alipaz <nicholas.alipaz@heremedia.com>

USER root

ENV MEMCACHED_VERSION 1.4.13

RUN apt-get install --no-install-recommends -y memcached=${MEMCACHED_VERSION}-* &&\
 apt-get clean

EXPOSE 11211
USER memcache
WORKDIR /

ENTRYPOINT ["/usr/bin/dumb-init", "--"]
CMD ["memcached"]
