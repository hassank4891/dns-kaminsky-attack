FROM ubuntu:24.04
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y bind9 tcpdump iproute2
COPY named.conf.options /etc/bind/named.conf.options
COPY named.conf.local /etc/bind/named.conf.local
CMD ["named", "-c", "/etc/bind/named.conf", "-g", "-u", "root"]