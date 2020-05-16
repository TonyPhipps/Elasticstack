- [Ubuntu](#ubuntu)
  - [Set Up NAT on a Virtual Switch](#set-up-nat-on-a-virtual-switch)
  - [Set Up Static IPs](#set-up-static-ips)
- [Elasticsearch](#elasticsearch)
  - [Install](#install)
  - [Enable Service, Run, and Verify](#enable-service-run-and-verify)
  - [Manage Service](#manage-service)
  - [Check Status and Monitor](#check-status-and-monitor)
  - [Configure](#configure)
- [Kibana](#kibana)
  - [Install](#install-1)
  - [Enable Service, Run, and Verify](#enable-service-run-and-verify-1)
  - [Manage Service](#manage-service-1)
  - [Check Status and Monitor](#check-status-and-monitor-1)
  - [Configure](#configure-1)
  - [Setup Index Patterns](#setup-index-patterns)
- [Logstash](#logstash)
  - [Install](#install-2)
  - [Enable Service, Run, and Verify](#enable-service-run-and-verify-2)
  - [Manage Service](#manage-service-2)
  - [Check Status and Monitor](#check-status-and-monitor-2)
  - [Configure](#configure-2)
- [WinLogBeat](#winlogbeat)
  - [Prepare LogStash for Receipt of Logs](#prepare-logstash-for-receipt-of-logs)
  - [Install](#install-3)
  - [Configure](#configure-3)
- [Filebeat](#filebeat)
  - [Install](#install-4)
  - [Configure](#configure-4)
  - [Start at Boot](#start-at-boot)
  - [Monitor](#monitor)
- [What Next?](#what-next)

This guide was written primarily with Ubuntu 18.04 LTS in mind, but can easily be adjusted to any other distribution. It is primarily comprised of condensed versions of the following guides:
- https://www.omgubuntu.co.uk/2018/09/hyper-v-ubuntu-1804-windows-integration
- https://www.elastic.co/guide/en/elasticsearch/reference/7.6/deb.html
- https://www.elastic.co/guide/en/kibana/7.1/install.html
- https://www.elastic.co/guide/en/logstash/7.1/installing-logstash.html
- https://www.elastic.co/guide/en/logstash/7.1/running-logstash.html
- https://www.elastic.co/guide/en/beats/winlogbeat/current/config-winlogbeat-logstash.html
- https://www.elastic.co/guide/en/beats/winlogbeat/master/logstash-output.html
- https://www.elastic.co/guide/en/beats/filebeat/master/filebeat-installation.html
- http://wiki.friendlyarm.com/wiki/index.php/Use_NetworkManager_to_configure_network_settings
- https://anandthearchitect.com/2018/01/06/windows-10-how-to-setup-nat-network-for-hyper-v-guests/

# Ubuntu
Do NOT select the option to log in automatically. This has been known to cause issues with the first logon. It won't cost you much time to test this, so feel free to ignore my advice to see if things have changed.

Install updates
```
sudo bash
apt-get update
apt-get upgrade
```

## Set Up NAT on a Virtual Switch

Open PowerShell and enter the following commands to set up a virtual switch
```
# Create Hyper-V internal only switch. 
New-VMSwitch –SwitchName “NAT-Switch” –SwitchType Internal –Verbose

# Note the IfIndex of NAT-Switch of the next command's output
Get-NetAdapter

# Create NAT Gateway
New-NetIPAddress –IPAddress 192.168.200.1 -PrefixLength 24 -InterfaceIndex ## –Verbose

# Assign each guest VM to this new adapter
Get-VM YourVMName | Get-VMNetworkAdapter | Connect-VMNetworkAdapter –SwitchName “NAT-Switch”

```

## Set Up Static IPs

Set up static IPs so your systems can point to each other properly. If you are running all services on one box, this and other related steps can be ignored or set to 127.0.0.1 // localhost.

Set your static IP. Use your home router IP as your DNS
```
nmcli connection modify 'Wired connection 1' connection.autoconnect yes ipv4.method manual ipv4.address 192.168.200.2/28 ipv4.gateway 192.168.200.1 ipv4.dns 192.168.1.1
reboot
```



# Elasticsearch

## Install
```
apt-get update
apt-get install curl
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
apt-get install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
apt-get update
apt-get install elasticsearch
```

## Enable Service, Run, and Verify
```
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable elasticsearch.service
apt-get install curl
curl -X GET localhost:9200
```

## Manage Service
```
systemctl stop elasticsearch.service
systemctl start elasticsearch.service
systemctl restart elasticsearch.service
```

## Check Status and Monitor
```
systemctl status elasticsearch.service
```
```
journalctl -u elasticsearch.service -f
```

## Configure
The following will allow Kibana to reach this server

Edit /etc/elasticsearch/elasticsearch.yml
```
network.host: 0.0.0.0
cluster.initial_master_nodes: ["node-1", "node-2"]
```
```
systemctl restart elasticsearch.service
```

# Kibana


## Install

```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
apt-get install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
apt-get update
apt-get install kibana
```



## Enable Service, Run, and Verify
```
/bin/systemctl daemon-reload
/bin/systemctl enable kibana.service
```
Allow the service a moment to start, then visit localhost:5601


## Manage Service
```
systemctl stop kibana.service
systemctl start kibana.service
systemctl restart kibana.service
```

## Check Status and Monitor
```
systemctl status kibana.service
```
```
journalctl -u kibana.service -f
```


## Configure
Edit /etc/kibana/kibana.yml
```
server.host: "0.0.0.0"
server.name: "kibana-vm"
elasticsearch.hosts:["http://elasticsearch-vm:9200"]
```
Ensure permissions on kibana folders and files are allowing the logstash account access
```
chown kibana:kibana /etc/kibana -R
chown kibana:kibana /var/log/kibana -R
chown kibana:kibana /var/lib/kibana -R
chown kibana:kibana /etc/default/kibana -R
```
```
systemctl restart kibana.service
```

## Setup Index Patterns
For each new event feed, you'll likely be setting up new Elasticsearch indices. For each one, you'll need to set up Index Patterns under Kibana's Management > Kibana > Index Patterns setup interface.

If you set up winlogbeat with mostly defaults, you'd use index pattern "winlogbeat*" with the @timestamp as your Time Filter.


# Logstash


## Install
```
apt-get install default-jre
update-alternatives --config java
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
apt-get install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
apt-get update
apt-get install logstash
```

Determine java location
```
update-java-alternatives --list
```

Append a JAVA_HOME environment variable
```
echo 'JAVA_HOME="/your/java/location"' | sudo tee -a /etc/environment
source /etc/environment
echo $JAVA_HOME
```


## Enable Service, Run, and Verify
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

## Check Status and Monitor
```
systemctl status logstash.service
```
```
/usr/share/logstash/bin/logstash --path.settings /etc/logstash --log.level=debug
journalctl -u logstash.service -f
```


## Configure

Edit /etc/logstash/logstash.yml
```
path.data: /var/lib/logstash
config.reload.automatic: true
config.reload.interval: 30s
log.level: warn
path.logs: /var/log/logstash
```

Ensure permissions on logstash folders and files are allowing the logstash account access
```
chown logstash:logstash /etc/logstash -R
chown logstash:logstash /var/log/logstash -R
chown logstash:logstash /var/lib/logstash -R
chown logstash:logstash /etc/default/logstash
```

# WinLogBeat

## Prepare LogStash for Receipt of Logs
Create /etc/logstash/conf.d/winlogbeat.yml:
```
input {
  beats {
    port => 5044
  }
}

output {
  elasticsearch {
    hosts => ["http://172.17.158.2:9200"]
    index => "%{[@metadata][beat]}-%{[@metadata][version]}" 
  }
}
```

## Install
https://www.elastic.co/downloads/beats/winlogbeat

## Configure
- Edit winlogbeat.yml. Comment out Elasticsearch lines and uncomment Logstash lines.

```
#-------------------------- Elasticsearch output ------------------------------
#output.elasticsearch:
  #hosts: ["localhost:9200"]

#----------------------------- Logstash output --------------------------------
output.logstash:
hosts: ["172.17.158.3:5044"]
```



# Filebeat


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

# What Next?
- [Ingest Meerkat Output](Meerkat.md)
- [Administration Tips & Tricks](https://github.com/TonyPhipps/Elasticstack/blob/master/Administration.md)
- [Enable HTTPS](HTTPS.md)