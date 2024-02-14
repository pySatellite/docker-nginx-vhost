# docker nginx vhost

![image](https://github.com/pySatellite/docker-nginx-vhost/assets/87309910/878eaf6a-18bc-4467-8b3f-5086de8ff3a1)

# load-balancing
- https://www.nginx.com/resources/glossary/load-balancing/

# step 1
- docker rm * rmi
```
$ sudo docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
$ sudo docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

# step 2
```
$ docker run -itd -p 8002:80 --name serv-a nginx
$ docker run -itd -p 8003:80 --name serv-b nginx
$ docker run -itd -p 8001:80 --name lb nginx:latest

$ sudo docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                                   NAMES
2fe6b3ebdb6e   nginx          "/docker-entrypoint.…"   5 seconds ago    Up 4 seconds    0.0.0.0:8002->80/tcp, :::8002->80/tcp   serv-a
fafba17b8cf6   nginx          "/docker-entrypoint.…"   18 seconds ago   Up 17 seconds   0.0.0.0:8003->80/tcp, :::8003->80/tcp   serv-b
fd3ca54a154a   nginx:latest   "/docker-entrypoint.…"   3 minutes ago    Up 3 minutes    0.0.0.0:8001->80/tcp, :::8001->80/tcp   lb
```

# step 3
```
$ sudo docker network ls
$ sudo docker network create abc
$ sudo docker network connect abc serv-a
$ sudo docker network connect abc serv-b
$ sudo docker network connect abc serv-c
```

# step 1,2,3 by Dockerfile
- build & push
```
$ tree
.
├── README.md
├── lb
│   ├── Dockerfile
│   └── config
│       └── default.conf
├── serv-a
│   ├── Dockerfile
│   └── index.html
└── serv-b
    ├── Dockerfile
    └── index.html

$ cd lb
$ sudo docker build -t pysatellite/lb:0.1.0 .
$ sudo docker push pysatellite/lb:0.1.0

$ cd ../serv-a
$ sudo docker build -t pysatellite/serv-a:0.1.0 .
$ sudo docker push pysatellite/serv-a:0.1.0

$ cd ../serv-b
$ sudo docker build -t pysatellite/serv-b:0.1.0 .
$ sudo docker push pysatellite/serv-b:0.1.0

```

- run
```
$ sudo docker network create -d bridge lb-net
$ sudo docker run -d --name serv-a --network lb-net pysatellite/serv-a:0.1.0
$ sudo docker run -d --name serv-b --network lb-net pysatellite/serv-b:0.1.0
$ sudo docker run -d -p 8001:80 --name lb --network lb-net pysatellite/lb:0.1.0
```

- test 1
```
$ sudo docker ps
CONTAINER ID   IMAGE                      COMMAND                  CREATED              STATUS              PORTS                                   NAMES
d86886fc9d9e   pysatellite/lb:0.1.0       "/docker-entrypoint.…"   3 seconds ago        Up 2 seconds        0.0.0.0:8001->80/tcp, :::8001->80/tcp   lb
e4444ea35c22   pysatellite/serv-b:0.1.0   "/docker-entrypoint.…"   About a minute ago   Up About a minute   80/tcp                                  serv-b
dd8af81cafb7   pysatellite/serv-a:0.1.0   "/docker-entrypoint.…"   About a minute ago   Up About a minute   80/tcp                                  serv-a
```

- test 2
```
$ curl http://localhost:8001
<h1>A</h1>
$ curl http://localhost:8001
<h1>B</h1>
$ curl http://localhost:8001
<h1>A</h1>
```

# ref
- https://hub.docker.com/
