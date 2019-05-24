
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
apt-get update && sudo apt-get install logstash
```

## Configure

Edit /etc/logstash/logstash.yml
```
- pipeline.id: main
path.config: "/etc/logstash/conf.d/*.conf"
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
/usr/share/logstash/bin/logstash -f /etc/logstash/logstash.yml -e
```