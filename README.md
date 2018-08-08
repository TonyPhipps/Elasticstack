## Ingest [THRecon](https://github.com/TonyPhipps/THRecon) into [Elasticstack](https://github.com/elastic)!

This guide was written primarily with [Xubuntu](https://xubuntu.org/about/) 18.04.1 in mind, but can easily be adjusted to any other distribution.

### Install Java
```
sudo apt-get update
sudo apt-get install -y python-software-properties software-properties-common apt-transport-https
sudo add-apt-repository ppa:webupd8team/java -y
sudo apt-get update
sudo apt-get install -y oracle-java8-installer
java -version
```


### Install Elasticsearch
```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo apt-get install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list
sudo apt-get update
sudo apt-get install elasticsearch
```

#### Configure ElasticSearch to listen on 9200
```
sudo mousepad /etc/elasticsearch/elasticsearch.yml
```

Remove comments on these lines
```
network.host: localhost
http.port: 9200
```

#### Configure ElasticSearch to run on startup and restart
```
sudo update-rc.d elasticsearch defaults 95 10
sudo -i service elasticsearch restart
```

After a minute, verify listening on port 5900
```
netstat -plntu
```

### Install Kibana
```
sudo apt-get install -y kibana
```

#### Configure Kibana
```
sudo mousepad /etc/kibana/kibana.yml &
```

Ucomment lines:
```
server.port: 5601
server.host: "localhost"
elasticsearch.url: "http://localhost:9200"
```

#### Configure Kibana to run on startup and restart
```
sudo update-rc.d kibana defaults 95 10
sudo -i service kibana restart
```

After a minute, verify listening on port 5601
```
netstat -plntu
```

### Install Nginx
(this can be skipped if you don't care to authenticate, i.e. in a lab environment)
```
sudo apt-get install -y nginx apache2-utils
```

#### Configure Nginx
```
sudo mkdir /etc/nginx/sites-avaiable
sudo /etc/nginx/sites-avaiable/touch kibana
sudo mousepad /etc/nginx/sites-available/kibana &
```

paste this:
```
server {
    listen 80;
 
    server_name 192.168.16.129;
 
    auth_basic "Restricted Access";
    auth_basic_user_file /etc/nginx/.kibana-user;
 
    location / {
        proxy_pass http://192.168.16.129:5601;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Create basic auth file and user
```
sudo htpasswd -c /etc/nginx/.kibana-user admin
TYPE YOUR PASSWORD
```

Activate the kibana virtual host
```
sudo ln -s /etc/nginx/sites-available/kibana /etc/nginx/sites-enabled/
```

Test the nginx configuration and make sure there is no error
```
sudo nginx -t
```

#### Configure Nginx to run on startup and restart
```
sudo update-rc.d nginx defaults 95 10
sudo -i service nginx restart
```

### Install Logstash
```
sudo apt-get install -y logstash
```

#### Configure output
Copy [threcon.conf](https://github.com/TonyPhipps/THRecon-Elasticstack/blob/master/Elasticstack6.3/threcon.conf) to /etc/logstash/conf.d/threcon.conf

#### Configure Logstash to run on startup and restart
```
sudo update-rc.d logstash defaults 95 10
sudo -i service logstash restart
```

### Install Filebeat
```
sudo apt-get install filebeat
```

#### Configure filebeat
```
mousepad /etc/filebeat/filebeat.yml
```

Comment out Elasticsearch Output section

Add:

```
#===================== External Input Configs ==============================
filebeat.config.inputs:
  enabled: true
  path: configs/*.yml
  reload.enabled: true
  reload.period: 10s
  
#========================= Logstash Outputs ================================
output.logstash:
  hosts: ["localhost:31337"]
```

Copy [threcon.yml](https://github.com/TonyPhipps/THRecon-Elasticstack/blob/master/Elasticstack6.3/threcon.yml) to /etc/logstash/configs/threcon.yml

### Configure Filebeat to run on startup and restart
```
sudo update-rc.d filebeat defaults 95 10
sudo -i service filebeat restart
```

#### Configuring filebeat with a VM Shared Folder
Create a symlink:
```
ln -s /mnt/hgfs/vm-share /var/log/threcon
```
Add this to filebeat.yml in the filebeat.inputs: section
```
  symlinks: true
```

### Sources
* https://www.elastic.co/guide/en/elastic-stack/current/installing-elastic-stack.html
* https://www.howtoforge.com/tutorial/ubuntu-elastic-stack/

