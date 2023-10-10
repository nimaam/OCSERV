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
RUN  cp /usr/src/ocserv/doc/sample.config /etc/ocserv/ocserv.conf 
WORKDIR /etc/ocserv
RUN rm -rf /usr/src/ocserv 
RUN runDeps="$(scanelf --needed --nobanner /usr/local/sbin/ocserv | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' | xargs -r apk info --installed | sort -u)" 
#
RUN apk add --virtual .run-deps $runDeps gnutls-utils iptables net-tools
RUN apk del .build-deps 
RUN rm -rf /var/cache/apk/*

RUN set -xe 
RUN  mkdir -p /etc/ocserv/certs 
RUN mkdir -p /etc/ocserv/config-per-user 
RUN  mkdir -p /etc/ocserv/config-per-group 
RUN mkdir -p /etc/ocserv/defaults 
RUN touch /etc/ocserv/defaults/user.conf 
RUN touch /etc/ocserv/defaults/group.conf 
RUN touch /etc/ocserv/ocpasswd 
RUN sed -i -e 's@\./sample.passwd@/etc/ocserv/ocpasswd@' \
              -e 's@\.\./tests/@/etc/ocserv/@' \
              -e 's@^#cert-group-oid =@cert-group-oid =@' \
              -e 's@^#compression =.*@compression = true@' \
              -e 's@^#config-per-@config-per-@' \
              -e 's@^#default-@default-@' \
              -e 's@^#enable-auth = "certificate"$@enable-auth = "certificate"@' \
              -e 's@^default-domain@#&@' \
              -e 's@^dns =.*@dns = 8.8.8.8@' \
              -e 's@^max-clients =.*@max-clients = 0@' \
              -e 's@^max-same-clients =.*@max-same-clients = 0@' \
              -e 's@^route@#&@' \
              -e 's@^try-mtu-discovery =.*@try-mtu-discovery = true@' \
              /etc/ocserv/ocserv.conf

COPY init.sh /init.sh
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
VOLUME /etc/ocserv
WORKDIR /etc/ocserv


EXPOSE 443/tcp 443/udp

ENTRYPOINT ["/entrypoint.sh"]
