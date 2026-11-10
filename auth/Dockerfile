FROM ubuntu:24.04
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y bind9 tcpdump
COPY named.conf.local /etc/bind/named.conf.local
COPY db.target.test /etc/bind/db.target.test
CMD ["named", "-c", "/etc/bind/named.conf", "-g", "-u", "root"]