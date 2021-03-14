
# Deploy Promtail


```
wget https://raw.githubusercontent.com/grafana/loki/v2.2.0/cmd/promtail/promtail-docker-config.yaml -O promtail-config.yaml
```{{execute}}



```
docker run -d --user=root -v /root/promtail-config.yaml:/usr/share/promtail-config.yaml -p 9080:9080 --name=promtail -v /var/log:/var/log grafana/promtail:2.2.0 -config.file=/usr/share/promtail-config.yaml
```{{execute}}