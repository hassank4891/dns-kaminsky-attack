FROM ubuntu:24.04
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y bind9 bind9-utils dnsutils tcpdump && rm -rf /var/lib/apt/lists/*
COPY named.conf /etc/bind/named.conf
COPY named.conf.options /etc/bind/named.conf.options
COPY named.conf.local /etc/bind/named.conf.local
COPY db.test /etc/bind/db.test
CMD ["named", "-g", "-c", "/etc/bind/named.conf"]
