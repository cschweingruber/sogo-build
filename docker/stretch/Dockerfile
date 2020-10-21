FROM debian:stretch as packages
LABEL maintainer="c.schweingruber@catatec.ch"
COPY build.sh .env ./
RUN ./build.sh

FROM phusion/baseimage:master-amd64

LABEL maintainer="c.schweingruber@catatec.ch"
# based on https://github.com/JensErat/docker-sogo
# repo from http://www.axis.cz/linux/debian/dists

COPY --from=packages /vendor /vendor

RUN apt update && \
    apt install -y dpkg-dev

#RUN dpkg-scanpackages /vendor | gzip > /vendor/Packages.gz

RUN echo "deb [trusted=yes] file:/// vendor/" > /etc/apt/sources.list.d/sogo.list

RUN apt update && \
    apt-get install -y --no-install-recommends gettext-base sogo sope4.9-gdl1-postgresql sope4.9-gdl1-mysql memcached tzdata && \
    apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Install Apache, SOGo from repository
#RUN echo "deb http://www.axis.cz/linux/debian xenial sogo-v4" > /etc/apt/sources.list.d/sogo.axis.cz.list && \
#    apt-key adv --keyserver pool.sks-keyservers.net --recv-key 608F5293A04BE668 && \
#    apt-get update && \
#    apt-get install -y --no-install-recommends gettext-base sogo sope4.9-gdl1-postgresql sope4.9-gdl1-mysql memcached tzdata && \
#    apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Move SOGo's data directory to /srv
RUN usermod --home /srv/lib/sogo sogo

# Fix memcached not listening on IPv6
RUN sed -i -e 's/^-l.*/-l localhost/' /etc/memcached.conf

# SOGo daemons
RUN mkdir /etc/service/sogod /etc/service/memcached
ADD sogod.sh /etc/service/sogod/run
ADD memcached.sh /etc/service/memcached/run

# Make GATEWAY host available, control memcached startup
RUN mkdir -p /etc/my_init.d
ADD gateway.sh memcached-control.sh /etc/my_init.d/

# Interface the environment
VOLUME /srv
VOLUME /usr/lib/GNUstep/SOGo/WebServerResources
EXPOSE 20000

# Baseimage init process
ENTRYPOINT ["/sbin/my_init"]
