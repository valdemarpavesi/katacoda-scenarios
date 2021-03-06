# Deploy  container-exporter

container level metrics

Verify config:

```
docker run -d --name PROM_CON_EXP \
              -p 9104:9104 \
              -v /sys/fs/cgroup:/cgroup \
              -v /var/run/docker.sock:/var/run/docker.sock \
              prom/container-exporter
```{{execute T2}}



If you're interested in seeing the raw metrics, they can be viewed with: 
```
curl localhost:9104/metrics
```{{execute T2}}



## Build a dashboard

In order to build a dashboard you can build one from scratch or you can use an existing one, for example:
https://grafana.com/grafana/dashboards/893

Let's use this existing dashboard. Copy the ID, and use the option `Import`.

`893`{{copy}}

Select **Prometheus** as the data source and Import.

![](import.png)

[View Dashboard for the targetCluster](https://[[HOST_SUBDOMAIN]]-3000-[[KATACODA_HOST]])

