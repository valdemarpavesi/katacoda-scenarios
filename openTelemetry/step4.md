
# Task

Now, we should generate traffic performing requests to the application.

Install h2load to generate http2 traffic:

`sudo apt install -y nghttp2-client`{{execute}}

Run traffic with 100 clients and 1000 requests:

`h2load http://localhost:8000/trace/1 -c 100 -n 1000 `{{execute}}


# TCPDUMP


```
docker inspect -f '{{.State.Pid}}' demo_jaeger-emitter_1
```{{execute}}

```
nsenter -t `docker inspect -f '{{.State.Pid}}' demo_jaeger-emitter_1` netstat -a -p
```{{execute}}

```
nsenter -t `docker inspect -f '{{.State.Pid}}' demo_jaeger-all-in-one_1` --net tcpdump udp port 6831
```{{execute}}
