FROM telegraf:1.36

# Enable non-free for snmp-mibs-downloader
RUN sed -i 's/Components: main/Components: main non-free/g' /etc/apt/sources.list.d/debian.sources 2>/dev/null || \
    sed -i 's/ main$/ main non-free/g' /etc/apt/sources.list 2>/dev/null || true

# Install base dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    apt-utils \
    apt-transport-https \
    curl \
    gnupg \
    snmp \
    snmp-mibs-downloader \
  && rm -rf /var/lib/apt/lists/*

# Setup Speedtest repo (new Ookla location)
RUN curl -fsSL https://packagecloud.io/ookla/speedtest-cli/gpgkey \
    | gpg --dearmor -o /usr/share/keyrings/speedtest.gpg \
  && echo "deb [signed-by=/usr/share/keyrings/speedtest.gpg] https://packagecloud.io/ookla/speedtest-cli/debian/ $(. /etc/os-release && echo $VERSION_CODENAME) main" \
    > /etc/apt/sources.list.d/speedtest.list \
  && apt-get update && apt-get install -y --no-install-recommends speedtest \
  && rm -rf /var/lib/apt/lists/*

# Disable MIB loading warnings
RUN sed -i 's/^mibs :/#mibs :/' /etc/snmp/snmp.conf 2>/dev/null || \
    echo "#mibs :" >> /etc/snmp/snmp.conf

EXPOSE 8125/udp 8092/udp 8094
ENTRYPOINT ["/entrypoint.sh"]
CMD ["telegraf"]
