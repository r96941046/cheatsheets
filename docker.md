# Docker Cheat Sheet

## References

1. https://www.docker.com/
2. http://docs.docker.com/mac/
3. https://www.openfoundry.org/tw/tech-column/9319-docker-101

## Install boot2docker for Mac

https://docs.docker.com/installation/mac/

- boot2docker creates Linux VM with virtualbox and control the docker daemon on Linux VM with the docker client on Mac
- `$(boot2docker shellinit 2> /dev/null)` put this in zshrc to set up environments for each session

## boot2docker vm port forwarding
if `docker run -d -p 9000:5000 training/webapp python app.py`
- boot2docker vm port forwarding: `VBoxManage controlvm boot2docker-vm natpf1 "webapp,tcp,127.0.0.1,9000,,9000"` 
- or run `boot2docker ip` and browse to the ip: `http://192.168.59.103:9000`

## Basics

```bash
docker login
docker version
docker info
docker search ubuntu
```

## Help
```bash
docker --help
docker run --help
```

## Run
download image if not present and start container
```bash
docker run hello-world
docker run docker/whalesay cowsay boo
```

docker run repo_name:tag_name
if tag_name is not provided, docker will default the tag_name to `latest`
```bash
docker run hello-world:latest
```

run container as a daemon (not exit after run command finished)
```bash
docker run -d ubuntu:12.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
```

run container as a daemon and map any required network ports inside our container to our host
mapped to a random high port within an ephemeral port range
```bash
docker run -d -P training/webapp python app.py
```

run container as a daemon and with custom port mapping (5000 in container to 80 in host)
this bind the port to all interfaces on the host machine
not a good idea, it constrains you to only one container on that specific port
```bash
docker run -d -p 80:5000 training/webapp python app.py
```

bind specific interface
```bash
docker run -d -p 127.0.0.1:80:5000 training/webapp python app.py
```

bind specific interface with a dynamic port
```bash
docker run -d -p 127.0.0.1::5000 training/webapp python app.py
```

bind UDP ports
```bash
docker run -d -p 127.0.0.0:80:5000/udp training/webapp python app.py
```

name the container
the name has to be unique, that means you can only call one container `web`
```bash
docker run -d -P --name web training/webapp python app.py
docker run -d --name db training/postgres
```

remove the container after it finishes running
cannot mix with daemon mode `-d`
```bash
docker run --rm hello-world
```

## Links
link to containers
`--link <name or id>:alias`

the source container `db` is not exposed in ports and secured, only exposes to recipient containers through:
1. environment variables
2. updating `/etc/hosts` file

```bash
docker run -d --name db training/postgres
docker run -d -P --name web --link db:db_alias training/webapp python app.py
```

## Env
list the specified container's environment variables
```bash
docker run --rm --name web2 --link db:db_alias training/webapp env
```

## Ports
specify the container name and the port in container to see the forwarded ports in host
```bash
docker port serene_bohr 5000
```

## Logs
see logs of container (use container id or container name)
```bash
docker logs 3ca34183b7df
```

## Inspect
see application in container's processes
```bash
docker top serene_bohr
```

see overview of container
```bash
docker inspect serene_bohr
```

extract information from the overview
```bash
docker inspect -f '{{ .NetworkSettings.IPAddress }}' serene_bohr
docker inspect -f "{{ .HostConfig.Links }}" web
```

## Volumes
1. data volumes are initialized when a container is created
2. data volumes can be shared and reused among containers
3. data volumes are designed to persist data, independent of the containers's life cycle, they're never automatically deleted
4. if the path already exists in the container the content will be replaced by the volume
5. you must explicitly call `docker rm -v` against the last container with a reference to the volume

custom volume
this can mount single file as well (not recommended)
```bash
docker run -t -i -v /myData centos:centos6 bash
docker run -d -P --name web -v /webapp training/webapp python app.py
```

custom volume (shared filesystem)
the folder /app in container is linked to the folder ~/apps/H2-Server in host
you can't do this with a Dockerfile because this is a host-dependent operation
```bash
docker run -tiv ~/apps/H2-Server/:/app ubuntu:12.04 bash
```

mount a directory read-only
```bash
docker run -d -P --name web -v /webapp:/opt/webapp:ro training/webapp python app.py
```

data volume container
share persistent data between containers
we can also extend the chain
```bash
docker create -v /dbdata --name dbdata training/postgres /bin/true
docker run -d --volumes-from dbdata --name db1 training/postgres
docker run -d --volumes-from dbdata --name db2 training/postgres
docker run -d --volumes-from db1 --name db3 training/postgres
```

remove volume containers
```bash
docker rm -v dbdata
```

use data volume container to backup and restore
```bash
docker run --volumes-from dbdata -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /dbdata
docker run -v /dbdata --name dbdata2 ubuntu /bin/bash
docker run --volumes-from dbdata2 -v $(pwd):/backup ubuntu cd /dbdata && tar xvf /backup/backup.tar
```

## Manage Containers
see list of active containers
```bash
docker ps
```

see list of all containers
```bash
docker ps -a
```

see the last container we started
```bash
docker ps -l
```

start container
```bash
docker start dc28027142e6
```

execute commands in active containers
```bash
docker exec -t -i dc28027142e6 bash
```

use exec to enter running containers
```bash
docker exec -ti dc28027142e6 bash
```

stop container (and thus stop the running container daemon)
```bash
docker stop
```

remove containers
```bash
docker rm dc28027142e6
```

## Manage Images
list images
```bash
docker images
```

list images with digest value
```bash
docker images --digests
```

remove image
```bash
docker rmi -f docker-whale
docker rmi -f dc28027142e6
```

## Dockerfile

- [Best practices for writing Dockerfiles](https://docs.docker.com/articles/dockerfile_best-practices/)

## Build
normal build
build with Dockerfile == a series of generating images and committing (intermidiate images and containers are later removed)
the dot refers to the path of Dockerfile, you can specify a custom Dockerfile path
```bash
docker build -t docker-whale .
```

build with tag
```bash
docker build -t="r96941046/docker-whale:version0.0.1" .
```

## Tag
```bash
docker tag dc28027142e6 r96941046/docker-whale:latest
```

## Push && Pull
```bash
docker push r96941046/docker-whale
docker pull r96941046/docker-whale
```

## Commit and make new image from modified container
commit is more cumbersome than `docker build`, use it instead
```bash
docker commit -m="Add some feature" -a="r96941046" dc28027142e6 r96941046/docker-whale:latest
```
