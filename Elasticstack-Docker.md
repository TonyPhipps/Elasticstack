This aims to install Elasticstack 6.3 via Docker on Xubuntu 18.04. YMMV.

## Install Docker from Official Repo
```
apt update
apt-get install curl
apt install apt-transport-https ca-certificates curl software-properties-common
echo 'deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable' >> /etc/apt/sources.list.d/docker.list
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
apt update
apt install docker-ce
```

## Install Docker-Compose
```
curl -L https://github.com/docker/compose/releases/download/1.20.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

## Pull Elasticstack Image
```
git clone https://github.com/elastic/stack-docker /user/share/elastic
sysctl -w vm.max_map_count=262144
```

## Prepare for install
Set the PWD Environment Variable
```
echo 'PWD=/usr/share/elastic/' >> /usr/share/elastic/.env
```

## Create Elasticstack Container
```
docker-compose -f .\setup.yml up
```
**SAVE THE PASSWORD PRESENTED AT THE END!**

### Run Container
```
docker-compose up -d
```

### Access Kibana/Elastic
```
infconfig eth0
```
Visit http://0.0.0.0:5601 for Kibana (with your eth0 IP address)

Username: kibana

Password: Given earlier

Visit http://0.0.0.0:9200 for Elastic (with your eth0 IP address)

Username: elastic

Password: Given earlier


### Enable HTTPS

set the following values in the following config files
- `/user/share/elastic/config/apm-server/apm-server.yml`
- `/user/share/elastic/config/auditbeat/auditbeat.yml`
- `/user/share/elastic/config/filebeat/filebeat.yml`
- `/user/share/elastic/config/heartbeat/heartbeat.yml`
- `/user/share/elastic/config/metricbeat/metricbeat.yml`
- `/user/share/elastic/config/packetbeat/packetbeat.yml`

```
setup.kibana:
host: "https://localhost:5601"
protocol: "https"
ssl.enabled: true
```

set the following values in `/user/share/elastic/config/kibana/kibana.yml`

```
server.ssl.enabled: true
server.ssl.certificate: /usr/share/kibana/config/certs/kibana/kibana.crt
server.ssl.key: /usr/share/kibana/config/certs/kibana/kibana.key
```

Then restart the stack

```
docker-compose restart
```

(Monitor status via `docker container ls`, and know that it takes a minute or so after containers are "healthy", and more time for Kibana to successfully connect to the elasticsearch service)