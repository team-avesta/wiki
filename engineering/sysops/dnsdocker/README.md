# Running a BIND server in Docker
[Source](https://github.com/sameersbn/docker-bind#sameersbnbind995-20170626 "Permalink to Dockerize BIND DNS server with webmin for DNS administration by sameersbn on github")


`Dockerfile` to create a [Docker](https://www.docker.com/) container image for [BIND](https://www.isc.org/downloads/bind/) DNS server bundled with the [Webmin](http://www.webmin.com/) interface.

BIND is open source software that implements the Domain Name System (DNS) protocols for the Internet. It is a reference implementation of those protocols, but it is also production-grade software, suitable for use in high-volume and high-reliability applications.


# Getting started

## Installation

Automated builds of the image are available on [Dockerhub](https://hub.docker.com/r/sameersbn/bind) and is the recommended method of installation.

> **Note**: Builds are also available on [Quay.io](https://quay.io/repository/sameersbn/bind)

```bash
docker pull sameersbn/bind:9.9.5-20170626
```

Alternatively you can build the image yourself.

```bash
docker build -t sameersbn/bind github.com/sameersbn/docker-bind
```

## Quickstart

Start BIND using:

```bash
docker run --name bind -d --restart=always \
  --publish 53:53/tcp --publish 53:53/udp --publish 10000:10000/tcp \
  --volume /srv/docker/bind:/data \
  sameersbn/bind:9.9.5-20170626
```
This command starts a docker container which listens for dns requests, on hosts ip on ports 53/tcp, 53/udp and 10000/tcp.
*Alternatively, you can use the sample [docker-compose.yml](https://github.com/sameersbn/docker-bind/blob/master/docker-compose.yml "source docker-compose.yml file") file to start the container using [Docker Compose](https://docs.docker.com/compose/)*

When the container is started the [Webmin](http://www.webmin.com/) service is also started and is accessible from the web browser at https://localhost:10000. Login to Webmin with the username `root` and password `password`. Specify `--env ROOT_PASSWORD=secretpassword` on the `docker run` command to set a password of your choosing.

The launch of Webmin can be disabled by adding `--env WEBMIN_ENABLED=false` to the `docker run` command. Note that the `ROOT_PASSWORD` parameter has no effect when the launch of Webmin is disabled.

Read the blog post [Deploying a DNS Server using Docker](http://www.damagehead.com/blog/2015/04/28/deploying-a-dns-server-using-docker/) for an example use case.

## Command-line arguments

You can customize the launch command of BIND server by specifying arguments to `named` on the `docker run` command. For example the following command prints the help menu of `named` command:

```bash
docker run --name bind -it --rm \
  --publish 53:53/tcp --publish 53:53/udp --publish 10000:10000/tcp \
  --volume /srv/docker/bind:/data \
  sameersbn/bind:9.9.5-20170626 -h
```

## Persistence

For the BIND to preserve its state across container shutdown and startup you should mount a volume at `/data`. We do this by default, but in case we have not done it to start with we can create the docker local system folder which will be mounted in the docker at /data using the following command.

> *The [Quickstart](#quickstart) command already mounts a volume for persistence.*


```bash
mkdir -p /srv/docker/bind

```

# Maintenance

## Upgrading

To upgrade to newer releases:

  1. Download the updated Docker image:

  ```bash
  docker pull sameersbn/bind:9.9.5-20170626
  ```

  2. Stop the currently running image:

  ```bash
  docker stop bind
  ```

  3. Remove the stopped container

  ```bash
  docker rm -v bind
  ```

  4. Start the updated image

  ```bash
  docker run -name bind -d \
    [OPTIONS] \
    sameersbn/bind:9.9.5-20170626
  ```

## Shell Access

For debugging and maintenance purposes you may want access the containers shell. If you are using Docker version `1.3.0` or higher you can access a running containers shell by starting `bash` using `docker exec`:

```bash
docker exec -it bind bash
```



# Test case uses

## Running the docker container on a separate ip address assigned to the same NIC. 

  To run a BIND docker on a host that already uses its ports upd/53 tcp/53 tcp/10000, we can either remap ports for either services that use those specific ports, or add a secondary IP address to our network card on which we forward the docker ports. To add a secondary Ip we can use legacy method "ifconfig" or the new method " ip addr". In our case we have used the legacy method. 
  We add a new temporary ip address to our Network Interface Card, and run our Docker on a separate IP address.[link](https://github.com/team-avesta/wiki/blob/master/engineering/devops/AddIP/README.md "Adding additional IP Addresses to a Network Interface Card on Ubuntu")
  In this example we added a temporary IP alias, and running docker on it. Any client in our network that requires dns services can access it from 192.168.1.50. 

```bash

ipconfig eth0:1 192.168.1.50

docker run --name bind -d --restart=always \
--publish 192.168.1.50:53:53/tcp \
--publish 192.168.1.50:53:53/udp \
--publish 192.168.1.50:10000:10000/tcp \
--volume /srv/docker/bind:/data sameersbn/bind:9.9.5-20170626

```

