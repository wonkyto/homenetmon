FROM telegraf:latest

ENV INSTALL_KEY 379CE192D401AB61

# Install snmp mib dependencies
RUN sed -i /etc/apt/sources.list -e 's/$/ non-free/g'

# Install dependencies for speedtest
RUN apt-get update && apt-get install -y apt-utils

RUN apt-get install -y gnupg1 apt-transport-https dirmngr

# Setup Speedtest repos
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys ${INSTALL_KEY}
RUN echo "deb https://ookla.bintray.com/debian" `grep VERSION_CODENAME /etc/os-release | sed -e 's/^.*=//'` "main" | tee /etc/apt/sources.list.d/speedtest.list

# Install Snmp Mibs and speedtest
RUN apt-get update && apt-get install -y \
  speedtest \
  snmp \
  snmp-mibs-downloader

# https://techexpert.tips/grafana/grafana-monitoring-snmp-devices/
# clearing the snmp config file
RUN sed -i /etc/snmp/snmp.conf -e 's/^mibs/#mibs/'

EXPOSE 8125/udp 8092/udp 8094

ENTRYPOINT ["/entrypoint.sh"]
CMD ["telegraf"]
