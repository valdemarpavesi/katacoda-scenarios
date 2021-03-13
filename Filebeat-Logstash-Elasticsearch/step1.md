# Verify Filebeat Logstash and Elasticsearh Performance

/var/log/* --> filebeat --> logstash --> elasticsearch

https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html


### Deploy Elasticsearch
```
docker run -d --net=host --name=elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.11.2
```{{execute}}


Verify:
```
curl -X GET "localhost:9200/_cat/nodes?v=true&pretty"
```{{execute}}

```
curl localhost:9200
```{{execute}}


Results:
`
ip         heap.percent ram.percent cpu load_1m load_5m load_15m node.role  master name
172.18.0.2           62          95  11    0.69    0.40     0.17 cdhilmrstw *      37afd33e70f0
`



### Deploy Logstash

Copy logstash.conf to /root/

```
cat << 'EOF' > /root/logstash.yml
input {
  beats {
    port => 5044
  }
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "%{[@metadata][beat]}-%{[@metadata][version]}" 
  }
}
EOF
```{{execute}}




logstash.yml
```
cat << 'EOF' > /root/logstash.yml
http.host: "0.0.0.0
xpack.monitoring.enabled: true
xpack.monitoring.elasticsearch.url: http://localhost:9200
EOF
```{{execute}}



Deploy logstash
```
docker run -d --rm -it \
--net=host --name=logstash -p 5044:5044  \
-v /root/logstash.conf:/usr/share/logstash/pipeline/logstash.conf \
-v /root/logstash.yml:/usr/share/logstash/config/logstash.yml \
docker.elastic.co/logstash/logstash:7.11.1
```{{execute}}


Verify:
```
docker ps -a
```{{execute}}

Config file:
```
docker exec logstash more /usr/share/logstash/pipeline/logstash.conf
```{{execute}}


```
docker logs logstash
```{{execute}}

Results:
<pre class="file">

[2021-03-13T02:22:50,015][INFO ][logstash.outputs.elasticsearch][main] Installing elasticsearch template to _template/logstash
[2021-03-13T02:22:50,964][INFO ][logstash.javapipeline    ][main] Pipeline Java execution initialization time {"seconds"=>1.01}
[2021-03-13T02:22:50,983][INFO ][logstash.inputs.beats    ][main] Starting input listener {:address=>"0.0.0.0:5044"}
[2021-03-13T02:22:51,001][INFO ][logstash.javapipeline    ][main] Pipeline started {"pipeline.id"=>"main"}
[2021-03-13T02:22:51,071][INFO ][logstash.agent           ] Pipelines running {:count=>1, :running_pipelines=>[:main], :non_running_pipelines=>[]}
[2021-03-13T02:22:51,161][INFO ][org.logstash.beats.Server][main][db5a67781e65c48adbecd85fbd1e99942978082594da35d277a9edd7cc8b5fa2] Starting server on port: 5044
[2021-03-13T02:22:51,305][INFO ][logstash.agent           ] Successfully started Logstash API endpoint {:port=>9600}

</pre>



type any + ctrl+c
```
nc localhost 5044
```{{execute}}

```
curl localhost:9600
```{{execute}}

```
curl -XGET 'localhost:9600/?pretty'
```{{execute}}

```
curl -XGET 'localhost:9600/_node/plugins?pretty'
```{{execute}}

```
curl -XGET 'localhost:9600/_node/hot_threads?pretty'
```{{execute}}

```
curl -XGET 'localhost:9600/_node/pipelines?pretty'
curl -XGET 'localhost:9600/_node/os?pretty'
curl -XGET 'localhost:9600/_node/jvm?pretty'
```{{execute}}

```
docker logs elasticsearch
```{{execute}}


Test from logstash to elasticsearch:
```
docker exec -it logstash curl localhost:9200
```{{execute}}



# Deploy Filebeat

https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-overview.html
https://www.elastic.co/guide/en/beats/filebeat/current/running-on-docker.html
https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-input-log.html
https://www.elastic.co/guide/en/beats/filebeat/current/logstash-output.html


Config file /root/filebeat.yml

```
cat << 'EOF' > /root/filebeat.yml
filebeat.inputs:
- type: log
  paths:
    - /var/log/messages
    - /var/log/*.log

output.logstash:
  hosts: ["localhost:5044"]
EOF
```{{execute}}


```
docker run -d --net=host --name=filebeat \
-v /root/filebeat.yml:/usr/share/filebeat/filebeat.yml \
docker.elastic.co/beats/filebeat:7.11.2 
```{{execute}}


Verify:
```
docker logs filebeat
```{{execute T2}}

Results:
`
2021-03-13T02:43:54.400Z        INFO    instance/beat.go:468    filebeat start running.
`


tcpdump
```
cnid=`docker ps | grep logstash |awk 'NR==1{print $1}'`
pid=`docker inspect -f '{{.State.Pid}}' $cnid`
echo $pid
nsenter -t $pid --net tcpdump tcp
```{{execute T2}}



netstat
```
cnid=`docker ps | grep logstash |awk 'NR==1{print $1}'`
pid=`docker inspect -f '{{.State.Pid}}' $cnid`
echo $pid
nsenter -t $pid netstat -s
nsenter -t $pid netstat -a -p

```{{execute T2}}