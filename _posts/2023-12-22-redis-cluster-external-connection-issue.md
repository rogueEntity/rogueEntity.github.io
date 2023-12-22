---
title: "Redis Cluster External Connection Issue"
excerpt: "redis cluster on docker made external connection issue"

date: 2023-12-22
last_modified_at: 2023-12-22

categories:
  - dev

tags:
  - Redis
  - dev

comments: true

toc: true
toc_sticky: true
---

## Intro

![redis_with_docker_01.jpg](/assets/images/posts/2023-12-22-redis-cluster-external-connection-issue/redis_with_docker_01.jpg){: .align-center}

↪ Nowadays, after market release of my team's first application ([TEFOMA](https://play.google.com/store/apps/details?id=com.peach_tri.tefoma&pli=1), assistant app for the board game, Terraforming Mars), we are on the way the original project, Iride-scent (Perfume Data Web). We tried to use Redis as an auth database.

And there was a tiny issue. Standalone mode Redis service had no issue, but after that, when I deployed Redis in cluster mode, it was unreachable on external tools that use redis client (like Datagrip). It still reachable with redis-cli on external environment, but it was impossible to connect on other tools. Fortunately, I could shoot the trouble. This is a little tip for who want to run **dockerized Redis cluster behind NAT/Port forwarding**.

---

## Why does this happen?

### no reachable nodes

![no_reachable_node_01.jpg](/assets/images/posts/2023-12-22-redis-cluster-external-connection-issue/no_reachable_node_01.jpg){: .align-center}

↪ It is why the phrase **'no reachable nodes'** is used that Redis client is impossible to detect any node to connect. That's what I thought at first, so I began to doubt whether the mapping between the docker and the host was done properly.

In addition, since this was a problem that did not occur when Redis was used as a standalone mode, I thought it was a problem specific to the Redis cluster and started to look for the cause.

The reason was that I did not know the additional necessary settings needed to run the Redis cluster in a docker container environment behind NAT/Port forwarding.

### Redis configuration

```plaintext
port ${each port number from node 1 to node 6}
cluster-enabled yes
cluster-node-timeout 5000
appendonly yes
protected-mode no
requirepass ${PW}
masterauth ${PW}
bind 0.0.0.0
```

↪ The first `docker-compose.yml` had seven containers. six services are for each cluster node, one to run redis-cli command to create a cluster. The network type was `bridge`. Redis configuration was like above. After searching some articles and posts, I've found out that I need additional properties.

The reason is that in Redis Cluster, clients gets the URLs of all Redis nodes from Redis node itself. So, in my case, "cluster nodes" request to one of the configured nodes. As a response, it receives the URL of all Redis nodes and tries to communicate with them. Since this is internal Docker network (cause it is the bridge driver), I get no reachable nodes in the cluster.

---

## How to fix

### redis.conf

```plaintext
cluster-announce-ip: The IP address to announce.
cluster-announce-port: The data port to announce.
cluster-announce-bus-port: The cluster bus port to announce.
```

↪ In Redis cluster, this problem can be solved with these additional configuration properties. These properties configure each Redis node to have a different "advertised" host and port. This way the clients will receive URL that they can access instead of the internal host and port.<sup>[1](#footnote_1)</sup>

---

## References

<a name="footnote_1">1</a>: https://stackoverflow.com/questions/57158537/can-not-connect-springboot-to-redis-cluster-on-docker

---