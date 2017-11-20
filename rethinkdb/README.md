#RethinkdDB Docker Cluster

Pull the docker image with ports exposed as 8080, 28015, 29015

```
docker pull zemuldo/rethinkdb:db1
```
Start Cluster Containers. We will run four of them. on ports

8081,29016,28016 --db1
8082,29017,28017 --db2
8083,29018,28018 --db3
8084,29019,28019 --db4

```
docker run -d -p 8081:8080 -p 29016:29015 -p 28016:28015 370c3d2c0222
```

```
docker run -d -p 8082:8080 -p 29017:29015 -p 28017:28015 370c3d2c0222
```

```
docker run -d -p 8083:8080 -p 29018:29015 -p 28018:28015 370c3d2c0222
```

```
docker run -d -p 8084:8080 -p 29019:29015 -p 28019:28015 370c3d2c0222
```

Find the container ID addresses of each container.


```
docker ps
```

For each run and get the ip address.

```
docker inspect 1275eb1b5404
```

Bind all the containers rethinkdb to all addresses to fully expose. and
Join the instances to the cluster Choose the primary one.

```
docker exec -d 1275eb1b5404 rethinkdb --bind all --join address:29015
```
Useful commands:

docker start | stop | restart container

docker exec -t -i container_name /bin/bash

