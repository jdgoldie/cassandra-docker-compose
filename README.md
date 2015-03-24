# A tiny multi-dc Cassandra cluster with Docker-compose

Yes, another Docker-Cassandra cluster...

### The Docker Image

The docker image is based off [abh1nav/cassandra|https://registry.hub.docker.com/u/abh1nav/cassandra/] 
with changes to support multiple data centers.  

Ops Center is currently disabled, but I hope to have time to get that working soon.

### The Cluster

Most of the Docker-Casssandra clusters are built with shell scripts that use a combination of 
IP-discovery of running containers and environment variables to link containers together.

I decided to try using [SkyDock|https://github.com/crosbymichael/skydock] and [SkyDNS|https://github.com/skynetservices/skydns1]
to handle service discovery.

SkyDock registers the IP for each docker container with SkyDNS using a simple, generated name.

In this cluster, I have the domain configured to be {{dev.docker}}.

### Docker-compose

[Docker-compose|https://docs.docker.com/compose/] is a nice way to orchestrate several containers 
from a single configuration file.  Starting this cluster is as simple as 

```
docker-compose -p cluster up -d 
```

The {{-p cluster}} speficies the cluster name.  This is important since compose will take the directory name by default.
To make the DNS names of the containers perdictable, we specify the name here.  This command should start the following:

```
cluster_seed2_1
cluster_nodedc2_1
cluster_nodedc1_1
cluster_seed1_1
cluster_skydns_1
cluster_skydock_1
```

This means that the assigned name for the {{seed1}} service is {{cluster_seed1_1.cassandra_compose.dev.docker}}. If you 
check the cassandra.yaml file, you will see it listed as one of the seed nodes.

Run {{docker run -t --rm jdgoldie/cassandra_compose watch -n 3 bin/nodetool -f cluster_seed1_1.cassandra_compose.dev.docker status}}
to see the status of the cluster.  Notice that, thanks to SkyDNS, the hostname resolves to the {{seed1}} container.

Now add some nodes.

```
docker-compose -p cluster scale nodedc1=2 nodedc2=2
```

Check {{nodetool}} again and you will see another node join each datacenter.

### Future

* Write a full blog entry to expand on docker-compose.yml
* Get Ops Center working 
* Use labels for datacenters instead of using environment variables.
* Sample application that writes to/from the cluster to demonstrate seamless failover.





