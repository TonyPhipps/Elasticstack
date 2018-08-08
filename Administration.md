### Monitoring

### View current listening ports
This is helpful when determining if stack services are listening properly.

`netstat -plntu`

### Monitor Filebeat

```
tail -f /var/log/filebeat/filebeat
```

OR.... run a single instance with INFO and ERROR sent straight to console
```
/usr/bin/filebeat -e -v
```

### Monitor Logstash
`tail -f /var/log/logstash/logstash-plain.log`

### Troubleshooting

#### "Unindexed Fields Cannot be Searched"
If Kibana is giving this error on some fields, you may have modified the fields in somewhere else in the chain (beats, logstash, elastic, etc)

`Kibana > Management > Select Index > Click Refresh Fields button in top right`

### Removing Indexes
Sometimes data doesn't ingest right and you just want to purge the index and reuse the name.
Remove index from Kibana via the web interface first, then remove from Elasticstack via web request:
```
curl -XDELETE localhost:9200/index/type/documentID
````
So `curl -XDELETE localhost:9200/logstash-bro` would remove the entire index
and `curl -XDELETE localhost:9200/windows*` would remove any index starting with windows