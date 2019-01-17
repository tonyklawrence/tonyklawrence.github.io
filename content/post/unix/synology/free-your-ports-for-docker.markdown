---
date: 2019-01-15
title: Free your Synology ports for Docker
slug: free-your-synology-ports

categories: [unix, synology]
keywords: “unix, nas, synology, docker”
draft: true
---

I’ve been running Pi-Hole on my Synology for a good few years.  It has taken me a while to figure out how to run it the way I liked to which is why I wrote the previous guide a few years ago.  (see: [Running Pi-Hole inside Docker on Synology](/post/unix/synology/running-pihole-inside-docker/))

Although this has helped me and many others, I was never quite happy about the outcome and have strived to find a better way.  I didn’t want to have to rely on WebStation or anything else outside of the docker container.  Thankfully I stumbled upon dockers network driver named macvlan.

> Note: I will be using the command line in this guide however the containers will still be visible in the Synology Docker UI.

### TL;DR (too long, don’t read)

Download the `docker-compose.yaml` file to your Synology and run the following command:

```
$sudo docker-compose up -d
```

Continue reading to understand (hopefully!)

### Using macvlan for networking

Macvlan is a network driver provided by Docker, the following is an extract from the [documentation](https://docs.docker.com/network/macvlan)

> Some applications, especially legacy applications or applications which monitor network traffic, expect to be directly connected to the physical network. In this type of situation, you can use the macvlan network driver to assign a MAC address to each container’s virtual network interface, making it appear to be a physical network interface directly connected to the physical network.

So the idea is that we will create our own docker network using the macvlan driver, this will then allow us to connect our Pi-Hole container onto this network which will assign it it’s own MAC.  This will then appear to be directly connected on our host network but will have it’s own IP and therefore all network ports available.

A new network can be easily created using the following command (but hold off, we’ll go into actually doing this later using configuration files)

``` bash
$sudo docker network create 
    —driver=macvlan 
    —gateway=192.168.123.1
    —subnet=192.168.123.0/24 
    —ip-range=192.168.1.200/28 
    -o parent=eth0
    name-of-network
```

> Note: The IP address range in the above command of 192.168.123.200 / 28 might appear strange.  This is so that we can restrict the IP addresses docker uses so not to clash with our physical machines [see IP calculator](http://jodies.de/ipcalc).

### Making things easier using docker compose

I’m quite a lazy guy when if comes to repitition.  I quickly became fed up of clicking around the docker UI to create containers, update containers and to modify them.  Luckily docker comes with a way to automate this setup with the command `docker-compose`.  I now use this to create all my containers that I run but in this example we will focus on Pi-Hole.

> Note: I’m using version `2` of the docker compose format.  This is the only version that seems to support specifying MAC and IP addresses which to me is very handy.  It only needs to be specified once at the top of each file.

Docker compose requires a configuration file that is in [YAML](https://yaml.org) format.  This is just plain text so can be edited using any application however, whitespace is important so no `TAB` characters please.

In the docker compose file we can create the network (macvlan), we can create our service (Pi-Hole) and optionally assign specific MAC and IP addresses.  I will explain each section individually to build up the complete picture.  I will also attach a download link at the bottom of the article so that you are not forced to copy & paste.

If you want to follow along then `ssh` to your Synology, create a file name `docker-compose.yaml` and add the following snippets. You can then try this out running the following command in the same place as the file you created.  This command will re-create the config each time so you do not need to delete previous versions.

``` bash
$sudo docker-compose up
```

> For more information try `sudo docker-compose up —help`

Start by specifying the version.

``` yaml
version: “2”                              # Version 2, only needed at top of each file
```

Defining the network:
``` yaml
networks:
  mynetwork:                              # Name of network
    driver: macvlan                       # Use the macvlan network driver
    driver_opts:
      parent: ovs_en0                     # If open vSwitch is disabled use en0 (or en1 +)
    ipam:
      config:
        - subnet: 192.168.123.0/24        # Specify subnet
          gateway: 192.168.123.254        # Gateway address
          ip-range: 192.168.123.200/28    # Available IP addresses
```

This creates a new network named `mynetwork` using the parent network interface `ovs_en0`.  This will be visible in the Synology docker UI under networks.

Next we need to add our Pi-Hole container.  This can be added with the following configuration.  I’m creating my own configuration for the hosts file and dnsmasq files (this allows for internal name resolution), this is optional.  I also create a volume for the pihole files so they are stored on my Synology instead of inside the container so that when I upgrade the statistics are maintained.

```yaml
services:
  pihole:
    container_name: pihole.         # We name our container here
    image: pihole/pihole:4.0.0-1    # Currently version 4.1 is not supported on Synology
    hostname: pihole                # Containers hostname (optional)
    domainname: my.network          # Contaners domain (optional)
    mac_address: d0:ca:ab:cd:ef:01  # Random MAC address (optional)
    networks:
      - mynetwork                   # Same name of network defined above
    ports:
      - 443/tcp
      - 53/tcp
      - 53/udp
      - 67/udp
      - 80/tcp
    environment:                    # Optional environment configuration
      ServerIP: 192.168.123.201
      WEBPASSWORD: “”
      VIRTUAL_HOST: pihole.my.network
      DNS1: 8.8.8.8
      DNS2: 8.8.4.4
    volumes:                        # Optional volume mounts
      - /volume1/docker/pihole/volume:/etc/pihole:rw
      - /volume1/docker/pihole/config/hosts:/etc/hosts:ro
      - /volume1/docker/pihole/config/resolv.conf:/etc/resolv.conf:ro
      - /volume1/docker/dns/config/dnsmasq.conf:/etc/dnsmasq.d/02-network.conf:ro
    restart: always                 # Set container to always restart
```

We specify our containers name, image and various networking information.  The environments required by Pi-Hole and any volumes we want to mount.  This is how I’ve configured my container but you could omit the volumes section if you don’t need this customisation.

The combination of these two configurations are all that is required to create and run Pi-Hole on your Synology NAS.  The complete file can be downloaded [here](/downloads/pihole/docker-compose.yaml)

If you need more information the documentation can be found [here](https://docs.docker.com/compose).

### How do we use docker compose

Assuming you have your docker compose file correctly setup (either writing your own or downloading one of mine) you can now start up Pi-Hole from the command line.  If all works out then Pi-Hole should now be up and running and visible inside the Synology Docker UI.

``` bash
$sudo docker-compose up -d
```

> Docker compose requires root access which will ask for your admin password.  The `-d` is needed to run in `daemon` mode, if we do not supply this then the command will block in the shell.

### How do we go about updating - you might ask?

Updating to the latest image is very easy.  You can run the following command.

``` bash
$sudo docker-compose pull
$sudo docker-compose up
```

> Optionally you can specify the service `$sudo docker-compose up pihole` if you have mulitple services

This will download the new version and then re-create the Pi-Hole container.

### Optional step; specify IP and one file per container

As I have many containers running on my Synology I separated each `service` into it’s own file and included it along with it’s configuration and volumes.  This way I can compose many docker files together rather than having one huge file.  I also specify the IP addresses for each container so that they never change.  This is accomplished by the following in my main `docker-compose.yaml` file (see [docker.zip](/downloads/pihole/docker.zip) for full example):

``` yaml
services:
  dns:
    extends:
      service: pihole
      file: pihole/docker-compose.yaml
    networks:
      home:
        ipv4_address: 192.168.123.200
```

Downloading the [`docker.zip`](/downloads/pihole/docker.zip) file will give you the mulitple docker files, or you can download the single [`docker-compose.yaml`](/downloads/pihole/docker-compose.yaml) file for a basic simple container.