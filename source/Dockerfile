FROM alpine:latest
MAINTAINER Nima Alizadeh <nima.alizadeh.m@gmail.com>

ENV OCSERV_URL https://gitlab.com/openconnect/ocserv/-/archive/master/ocserv-master.tar.gz

RUN apk update
RUN apk add curl git wget g++ gnutls-dev gpgme libev-dev libnl3-dev libseccomp-dev linux-headers linux-pam-dev lz4-dev make readline-dev tar xz 

RUN mkdir -p /usr/src/ocserv
WORKDIR /usr/src/
RUN curl -SL https://www.infradead.org/ocserv/download/ocserv-1.2.2.tar.xz -o ocserv.tar.xz
RUN tar -xvf ocserv.tar.xz -C /usr/src/ocserv --strip-component=1
WORKDIR /usr/src/ocserv
RUN ./configure 
RUN make 
RUN  make install
RUN mkdir -p /etc/ocserv 
RUN rm -rf /ocserv.conf
WORKDIR /etc/ocserv
RUN rm -rf /usr/src/ocserv 
RUN runDeps="$(scanelf --needed --nobanner /usr/local/sbin/ocserv | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' | xargs -r apk info --installed | sort -u)" 
#
RUN apk add --virtual .run-deps $runDeps gnutls-utils iptables net-tools linux-pam libev libseccomp lz4-libs certbot
RUN apk del curl git wget g++ gnutls-dev gpgme libev-dev libnl3-dev libseccomp-dev linux-headers linux-pam-dev lz4-dev make readline-dev tar xz
RUN rm -rf /var/cache/apk/*

RUN set -xe 
RUN mkdir -p /etc/ocserv/config-per-user 
RUN  mkdir -p /etc/ocserv/config-per-group 
RUN mkdir -p /etc/ocserv/defaults 
RUN touch /etc/ocserv/defaults/user.conf 
RUN touch /etc/ocserv/defaults/group.conf 
COPY conf/ocserv.conf /etc/ocserv.conf
COPY init.sh /init.sh
RUN chmod +x /init.sh
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
VOLUME /etc/ocserv
WORKDIR /etc/ocserv


EXPOSE 443/tcp 443/udp

ENTRYPOINT ["/entrypoint.sh"]
