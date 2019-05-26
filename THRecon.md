This guide explains how to use Filebeat to harvest THRecon output and send it to Logstash. To install the Elasticstack, see [README.md](README.md)

# Logstash

Copy ```Logstash/THRecon.yml``` to ```/etc/logstash/conf.d```


## Restart Logstash Service

Note: not required with Logstash option ```config.reload.automatic: true```
```
systemctl restart logstash.service
```

# Filebeat

- Paste the contents of ```Filebeat/threcon.yml``` into ```filebeat.yml``, immediately after this line:
```
filebeat.inputs:
```
