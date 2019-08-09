- [Elasticsearch](#elasticsearch)
  - [Install](#install)
  - [Configure](#configure)
  - [Start at Boot](#start-at-boot)
  - [Manage Service](#manage-service)
  - [Check Status](#check-status)
- [Kibana](#kibana)
  - [Install](#install-1)
  - [Configure](#configure-1)
  - [Start at Boot](#start-at-boot-1)
  - [Manage Service](#manage-service-1)
- [Logstash](#logstash)
  - [Install](#install-2)
  - [Configure](#configure-2)
  - [Start at Boot](#start-at-boot-2)
  - [Manage Service](#manage-service-2)
  - [Monitor](#monitor)
- [Filebeat](#filebeat)
  - [Install](#install-3)
  - [Configure](#configure-3)
  - [Start at Boot](#start-at-boot-3)
  - [Monitor](#monitor-1)
- [Ingest Meerkat Output](#ingest-meerkat-output)
- [Administration Tips & Tricks](#administration-tips--tricks)
- [Enable HTTPS](#enable-https)

This guide was written primarily with [Xubuntu](https://xubuntu.org/about/) 18.04.1 in mind, but can easily be adjusted to any other distribution.


# Elasticsearch
Condensed version of the below guide compiled using Xubuntu

https://www.elastic.co/guide/en/elasticsearch/reference/7.1/deb.html#deb-repo

## Install
```
apt-get update
apt-get install curl
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
apt-get install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
apt-get update && sudo apt-get install elasticsearch
```

## Configure
Edit /etc/elasticsearch/elasticsearch.yml
```
network.host: 0.0.0.0
cluster.initial_master_nodes: ["node-1", "node-2"]
```

## Start at Boot
```
/bin/systemctl daemon-reload
/bin/systemctl enable elasticsearch.service
```

## Manage Service
```
systemctl stop elasticsearch.service
systemctl start elasticsearch.service
systemctl restart elasticsearch.service
```

## Check Status
```
systemctl status elasticsearch.service
curl -X GET http://127.0.0.1:9200
```

# Kibana
Condensed version of the below guide compiled using Xubuntu

https://www.elastic.co/guide/en/kibana/7.1/install.html

## Install
```
apt-get update && sudo apt-get install kibana
```

## Configure
Edit /etc/kibana/kibana.yml
```
server.host: "0"
```

## Start at Boot
```
/bin/systemctl daemon-reload
/bin/systemctl enable kibana.service
```

## Manage Service
```
systemctl stop kibana.service
systemctl start kibana.service
systemctl restart kibana.service
```

# Logstash
Condensed version of the below guides compiled using Xubuntu

https://www.elastic.co/guide/en/logstash/7.1/installing-logstash.html

https://www.elastic.co/guide/en/logstash/7.1/running-logstash.html

## Install
```
apt-get install default-jre
update-alternatives --config java
apt-get update && sudo apt-get install logstash
```

Determine java location
```
update-java-alternatives --list
```

Edit /etc/environment
```
JAVA_HOME="/your/java/location"
```
```
source /etc/environment
echo $JAVA_HOME
```

## Configure

Edit /etc/logstash/logstash.yml
```
path.data: /var/lib/logstash
path.logs: /var/log/logstash
path.config: /etc/logstash
config.reload.automatic: true
config.reload.interval: 30s
log.level: warn
```

Ensure permissions on logstash folders and files are allowing the logstash account access
```
chown logstash:logstash /etc/logstash -R
chown logstash:logstash /var/log/logstash -R
chown logstash:logstash /var/lib/logstash -R
chown logstash:logstash /etc/default/logstash
```

## Start at Boot
```
/bin/systemctl daemon-reload
/bin/systemctl enable logstash.service
```

## Manage Service
```
systemctl stop logstash.service
systemctl start logstash.service
systemctl restart logstash.service
```

## Monitor
```
/usr/share/logstash/bin/logstash --path.settings /etc/logstash --log.level=debug
```

# Filebeat
Condensed version of the below guides compiled using Xubuntu

https://www.elastic.co/guide/en/beats/filebeat/master/filebeat-installation.html

## Install

- Download and install Filebeat to a windows box according to the guide

## Configure

- Edit filebeat.yml and place input config immediately after this line:
```
filebeat.inputs:
```

Edit filebeat.yml, comment out Elasticsearch section and uncomment Logstash section.
```
#output.elasticsearch:
  # Array of hosts to connect to.
#  hosts: ["localhost:9200"]

output.logstash:
  # The Logstash hosts
  hosts: ["172.30.211.88:9600"]
```

## Start at Boot

Ensure the "filebeat" service was created via services.msc

## Monitor

Either:
- Review the logs at c:\programdata\filebeat\logs
- run ```..\filebeat.exe -e -v``` and review output to console

# [Ingest Meerkat Output](Meerkat.md)
# [Administration Tips & Tricks](https://github.com/TonyPhipps/meerkat-Elasticstack/blob/master/Administration.md)
# [Enable HTTPS](HTTPS.md)