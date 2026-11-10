FROM ubuntu:24.04
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y \
    python3 \
    python3-pip \
    dnsutils \
    tcpdump \
    iproute2 \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY . /app

# Install tool requirements (use --break-system-packages for Ubuntu 22.04+ pip)
RUN pip3 install --break-system-packages scapy blessings dnspython

# Keep container running so we can exec into it and run the attack manually
CMD ["sleep", "infinity"]