# Home Network Monitoring
Here we will try to set up a system to monitor our network environment.

We will use 3 main technologies:
- telegraf (collection)
- influxdb (database)
- grafana (frontend)

## Requirements
This requires you to have:
- Docker
- Docker Compose

## Starting the system
Generally everything will need to be owner/group owned by root, except the grafana subdirectory, which should be owned by 472, and is described down in the configuation of grafana below.

Once you have configured everything (see below) simply run:
```
docker-compose up -d
```

## Configuration
### Telegraf
We are using a modified version of the offical telegraf docker container. This new version contains:
- the offical Ookla Speedtest cli command
- snmp mib files

This container can be built by doing the following
```
cd Dockerfiles/telegraf
docker build wonkyto/telegraf ./
```
#### Generating speedtest license file
In order to use the speedtest command you will need to run the speedtest cli command a first time and agree to the license to generate a license file. This can be done by the following:

```
docker run --rm -v speedtest/.config:/root/.config wonkyto/telegraf /usr/bin/speedtest --accept-license
```

This will create a file in: speedtest/.config/ookla/speedtest-cli.json

#### Generating a basic telegraf config file
*Note!*: This is not needed, it's just included for knowledge
For the purposes of completeness we include the process for generating a minimal telegraf configuration file. This is not necessary as we already include a basic file which has already been modified to communicate with it's sister influxdb container. This is located in telegraf/telegraf.conf

```
docker run --rm telegraf telegraf config > telegraf.conf
```

#### Other changes
We have modified the basic config (telegraf/telegraf.conf) to include the following changes:
- endpoint for influxdb: influxdb (defined in the docker-compose.yml)
- name of the influxdb database our data will be stored: telegraf
```
   urls = ["http://influxdb:8086"]
   database = "telegraf"
```

#### Configuring Speedtest
The following block can be placed inside the telegraf.conf
Please see telegraf/telegraf.conf.example for the following section:
```
# Exec a speedtest every 5 minutes
# Required: speedtest from ookla
[[inputs.exec]]
  interval = "300s"

  # Shell/commands array
  # Full command line to executable with parameters, or a glob pattern to run all matching files.
  commands = ["/usr/bin/speedtest -f json"]

  ## Timeout for each command to complete.
  timeout = "60s"

  # Data format to consume.
  # NOTE json only reads numerical measurements, strings and booleans are ignored.
  data_format = "json"
  json_string_fields = ["interface_externalIp", "interface_internalIp", "isp", "result_url"]
  # measurement name suffix (for separating different commands)
  name_suffix = "_speedtest"
```
#### Configuring SNMP
The following block can be placed inside the telegraf.conf.

What's most important here are the *agents* and the *community* fields. You will needs to add an entry for each of your agents in the json array. For the community string, place in the shared community string for authentication. 

If you have multiple devices which have different community strings, you can duplicate this block with differing values.

The rest of the config block does a few things:
- Gets the hostname (makes it a tag)
- Gets the uptime (This is in timeticks which are 10s of milliseconds - gotcha when it comes to making a grafana visualisation - remember to divide by 100 and use seconds or multiple by 10 and use milliseconds)
- Interface Stats + interface names
- High Capacity Interface stats + High Capcity Interface Names

These oids/mibs are general enough that it appears to work on most network devices/linux.

Please see telegraf/telegraf.conf.example for the following section:
```
[[inputs.snmp]]
  agents = [ "192.168.0.1", "192.168.0.2", "192.168.0.4", "192.168.0.5" ]
  version = 2
  community = "public"
  interval = "60s"

  [[inputs.snmp.field]]
    name = "hostname"
    oid = "RFC1213-MIB::sysName.0"
    is_tag = true

  [[inputs.snmp.field]]
    name = "uptime"
    oid = "DISMAN-EXPRESSION-MIB::sysUpTimeInstance"

  # IF-MIB::ifTable contains counters on input and output traffic as well as errors and discards.
  [[inputs.snmp.table]]
    name = "interface"
    inherit_tags = [ "hostname" ]
    oid = "IF-MIB::ifTable"

    # Interface tag - used to identify interface in metrics database
    [[inputs.snmp.table.field]]
      name = "ifDescr"
      oid = "IF-MIB::ifDescr" # This one is the old names, which only have eth0/eth1 etc, not WAN WAN2 etc
      is_tag = true

  # IF-MIB::ifXTable contains newer High Capacity (HC) counters that do not overflow as fast for a few of the ifTable counters
  [[inputs.snmp.table]]
    name = "interface"
    inherit_tags = [ "hostname" ]
    oid = "IF-MIB::ifXTable"

    # Interface tag - used to identify interface in metrics database
    [[inputs.snmp.table.field]]
      name = "ifDescr"
      oid = "IF-MIB::ifXTable.1.18"
      is_tag = true
```

### Influxdb
We have not made any changes to the influxdb configuration, and are using the basic configuration that influxdb will generate itself. To make things easier we are distributing the same content, but you could generate this yourself

#### Generating a basic influxdb config file
*Note!*: This is not needed, it's just included for knowledge
For the purposes of completeness the process for generating a minimal influxdb configuration file is included. This is unnecessary as a basic configuration file has been included. This is located in influxdb/influxdb.conf

```
docker run --rm telegraf telegraf config > telegraf.conf
```

### Grafana
Initial configuration of grafana is done with the docker-compose.yml using environment variables.

You must also chown the grafana subdirectory to the user 472
```
chown -R 472:472 grafana
```

The rest of the configuration must be done via the gui, once you've started the containers - see starting above.

#### Setting up a datasource
In order to create your dashboards you must set up a data source, which will be the influxdb instance
* Click: Cog -> Data sources
* Click: Add Data source
* Click: Influxdb
* Add http://influxdb:8086 into the HTTP URL field
* Click Save and Test

Keep in mind we haven't set up any TLS certs or authentication.

#### Dashboards
For the purposes of getting started, an exported json of the dashboards has been included as a starting point. This is located in grafana-dashboards/dashboard-example.json.

You can import this by:
* Hover over the +
* Click Import
* Either Click: Upload .json file, or paste content of the example file into the paste field.
# homenetmon
