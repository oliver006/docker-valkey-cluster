# docker-valkey-cluster

Docker image for LOCAL development with a valkey cluster based on the published valkey docker image.

Forked from [mix3](https://github.com/mix3/valkey-redis-cluster) and modified to add support for `VALKEY_PASSWORD` and other features.
This is mainly used for CI for the [redis_exporter project](https://github.com/oliver006/redis_exporter).


***Note:*** This is _not_ intended for production use; it runs 6 nodes of a valkey cluster in a single container intended as a convenience for local development and testing of code that uses a real valkey cluster (i.e. AWS Elasticache/Valkey, etc.).  

Why not just run a single node for local development?  Because you'll miss bugs in your code (i.e. sending a pipeline of commands that reference keys that reside in different nodes) that only show up when you have a cluster.  


For a current list of available Docker image tags see: https://hub.docker.com/r/oliver006/valkey-cluster/tags



# Important for Mac users

If you are using this container to run a valkey cluster on your mac computer, then you need to configure the container to use another IP address for cluster discovery as it can't use the default discovery IP that is hardcoded into the container.

If you are using the docker-compose file to build the container, then you must export a environment variable on your machine before building the container.

```
# This will make valkey do cluster discovery and bind all nodes to ip 127.0.0.1 internally

export VALKEY_CLUSTER_IP=0.0.0.0
```

If you are downloading the container from dockerhub, you must add the internal IP environment variable to your `docker run` command.

```
docker run -e "IP=0.0.0.0" -p 7000-7005:7000-7005 oliver006/valkey-cluster:latest
```



# Usage


## Include sentinel instances

Sentinel instances is not enabled by default.

If running with plain docker send in `-e SENTINEL=true`.

When running with docker-compose set the environment variable on your system `VALKEY_USE_SENTINEL=true` and start your container.

    services:
      valkey-cluster:
        ...
      environment:
        SENTINEL: 'true'


## Change number of nodes

Be default, it is going to launch 3 masters with 1 slave per master. This is configurable through a number of environment variables:

| Environment variable | Default |
| -------------------- |--------:|
| `INITIAL_PORT`       |    7000 |
| `MASTERS`            |       3 |
| `SLAVES_PER_MASTER`  |       1 | 

Therefore, the total number of nodes (`NODES`) is going to be `$MASTERS * ( $SLAVES_PER_MASTER  + 1 )` and ports are going to range from `$INITIAL_PORT` to `$INITIAL_PORT + NODES - 1`.

At the docker-compose provided by this repository, ports 7000-7050 are already mapped to the hosts'. Either if you need more than 50 nodes in total or if you need to change the initial port number, you should override those values.

Also note that the number of sentinels (if enabled) is the same as the number of masters. The docker-compose file already maps ports 5000-5010 by default. You should also override those values if you have more than 10 masters.

    services:
      valkey-cluster:
        ...
      environment:
        INITIAL_PORT: 9000,
        MASTERS: 2,
        SLAVES_PER_MASTER: 2


## IPv6 support

By default, valkey instances will bind and accept requests from any IPv4 network.
This is configurable by an environment variable that specifies which address a valkey instance will bind to.
By using the IPv6 variant `::` as counterpart to IPv4s `0.0.0.0` an IPv6 cluster can be created.

| Environment variable | Default |
| -------------------- | ------: |
| `BIND_ADDRESS`       | 0.0.0.0 |

Note that Docker also needs to be [configured](https://docs.docker.com/config/daemon/ipv6/) for IPv6 support.
Unfortunately Docker does not handle IPv6 NAT so, when acceptable, `--network host` can be used.

    # Example using plain docker
    docker run -e "IP=::1" -e "BIND_ADDRESS=::" --network host oliver006/valkey-cluster:latest


## Build alternative valkey versions


### docker build

My github actions use docker buildx to build a linux/arm64 and linux/amd64 image for the current valkey version. That image is pushed to `docker hub` as oliver006/valkey-cluster:<valkey-version>-<git tag> and oliver006/valkey-cluster:latest.


# License

This repo is using the MIT LICENSE.  And is based on the original work by Johan Andersson in grokzen/redis-cluster.  

You can find it in the file [LICENSE](LICENSE)
