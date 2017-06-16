# Infrasturcture Monitoring with Docker

## Note:- There should be realatively easiest approach when your run Docker Enginer on Swarm mode. This solution is only applicable when you wants to monitor standalone docker engine.

## Components Use:-
1. Consul
2. Prometheus
3. Node-Exporter
4. Registrator
5. Grafana

## Flow: -
 
Everything is working here as a Docker Container.
 
We are using **Consul(Standalone Cluster)** as a service discovery tools which register each node that is going to monitor by Prometheus.
**Node-Exporter** is run as a container on each node that will gather all system related matrix.
**Prometheus** will works on pull model so it will query consul to get the node details and based on available result it will scrape matrix from Node-Exporter. We can also create alert rule on Prometheus so it will alert on email or slack when predefine event has been occurs.
**Registrator** is deploy as a container on every node so it will keep track on each services state that should be monitor by Prometheus.
**Grafana** is providing Graphs UI where we can create multiple dashboard and best way to understand what’s going on in our system. Grafana is connect with Prometheus data source to get data to be ploted on graphs.


# Run Consul standalone container on your monitoring machine
```
docker run -d -h node \
 --name=consul-cluster \
 -p 8500:8500 -p 8600:53/udp \
  progrium/consul \
  -server -bootstrap \
  -log-level debug

```

# Run Prometheus on your monitoring machine
```
docker run -p 9090:9090 \
 -d -v $PWD/prometheus.yml:/etc/prometheus/prometheus.yml \
 --name prometheus \
 prom/prometheus:v1.6.3

```

# Run Grafana on your monitoring machine
```
docker run -d \
 -p 3000:3000 \
 --name grafana \
 grafana/grafana

```

# Run Registrator on every machine
```
docker run -d \
 --name=registrator \
 --net=host \
 --volume=/var/run/docker.sock:/tmp/docker.sock \
 gliderlabs/registrator:latest \
 -ip="<Local_IP_Address>" \
 consul://<Consul_IP_Address>:8500

```


# Run Node-Exporter on every machine

```
docker run -d \
 -p 9100:9100 \
 -v /proc:/host/proc \
 -v /sys:/host/sys \
 -v /:/rootfs \
 prom/node-exporter \
 -collector.procfs=/host/proc \
 -collector.sysfs=/host/proc \
 -collector.filesystem.ignored-mount-points="^/(sys|proc|dev|host|etc)($$|/)"

```