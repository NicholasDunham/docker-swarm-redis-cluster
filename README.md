# Running Redis Cluster on Docker Swarm

### Get a swarm discovery token.
```
token=$(docker run swarm create)
```

### Create the swarm master machine.
```
docker-machine create -d virtualbox --swarm --swarm-master --swarm-discovery token://$token swarm-master
```

### Create six Redis machines to cluster.
```
docker-machine create -d virtualbox --swarm --swarm-discovery token://$token redis-1
docker-machine create -d virtualbox --swarm --swarm-discovery token://$token redis-2
docker-machine create -d virtualbox --swarm --swarm-discovery token://$token redis-3
docker-machine create -d virtualbox --swarm --swarm-discovery token://$token redis-4
docker-machine create -d virtualbox --swarm --swarm-discovery token://$token redis-5
docker-machine create -d virtualbox --swarm --swarm-discovery token://$token redis-6
```

### List the machines we've created so far
```
docker-machine ls
```

### Set docker-machine and the local shell to use the swarm master we created earlier.
```
docker-machine env --swarm swarm-master
eval $(docker-machine env --swarm swarm-master)
```

### Create the Redis instances to cluster. All paths, including the path to the local copy of the redis.conf file, must be absolute. The --net=host option is necessary for Redis Cluster's discovery to work.
```
docker run -v <PATH_TO>/redis.conf:/usr/local/etc/redis/redis.conf -d --net=host --name redis1 redis redis-server /usr/local/etc/redis/redis.conf
docker run -v <PATH_TO>/redis.conf:/usr/local/etc/redis/redis.conf -d --net=host --name redis2 redis redis-server /usr/local/etc/redis/redis.conf
docker run -v <PATH_TO>/redis.conf:/usr/local/etc/redis/redis.conf -d --net=host --name redis3 redis redis-server /usr/local/etc/redis/redis.conf
docker run -v <PATH_TO>/redis.conf:/usr/local/etc/redis/redis.conf -d --net=host --name redis4 redis redis-server /usr/local/etc/redis/redis.conf
docker run -v <PATH_TO>/redis.conf:/usr/local/etc/redis/redis.conf -d --net=host --name redis5 redis redis-server /usr/local/etc/redis/redis.conf
docker run -v <PATH_TO>/redis.conf:/usr/local/etc/redis/redis.conf -d --net=host --name redis6 redis redis-server /usr/local/etc/redis/redis.conf
```

### List running instances.
```
docker ps
```

### More verbose information about running instances--take note of the IP
### addresses our Redis intances have been assigned.
```
docker info
```

### redis-trib manages the cluster for us. We'll download it from the Redis GitHub repo.
```
curl -O https://raw.githubusercontent.com/antirez/redis/unstable/src/redis-trib.rb
```

### We'll also need the Redis Ruby gem.
```
gem install redis
```

### Now we can create the cluster from the Redis instances we created earlier.
```
./redis-trib.rb create --replicas 1 <redis1_IP>:6379 <redis2_IP>:6379 <redis3_IP>:6379 <redis4_IP>:6379 <redis5_IP>:6379 <redis6_IP>:6379
```

### Finally, use the Redis CLI client to connect to one of the Redis instances
### and test the cluster.
```
redis-cli -c -h <redis1_IP> -p 6379
```
