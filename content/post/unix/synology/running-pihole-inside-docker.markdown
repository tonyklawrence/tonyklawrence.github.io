---
date: 2017-11-29T13:37:28Z
title: Running Pi-Hole inside Docker on Synology
slug: running-pihole-inside-docker

categories: [unix, synology]
keywords: “unix, nas, synology, docker, pihole”
---
> **Update** This post was updated in January 2018 and details how to get the Debian version of pihole-docker running as the Alpine version is no longer supported.

When I first wrote about installing Pi-Hole inside Docker on my Synology NAS I came up with a solution that required a little modification to the standard DSM (see: [Freeing up port 80 on Synology DSM](/post/unix/synology/freeing-port-80/)).  Whilst this worked I was never completely happy with this approach as I never want to modify system files as you can never be sure.

After a little work and a few updates to the Pi-Hole docker image I feel this is now possible without modification.  Below is how I achieve this, enjoy.

> All information relating to the pi-hole docker image and extra configuration can be found on it’s home page. https://github.com/diginc/docker-pi-hole

### Running Pi-Hole

I originally used the Alpine version of this image due to it being smaller, however, as this is no longer maintained we will install the latest Debian version.  Once you have created your container from the latest tagged image there are a few steps in the docker wizard that are important.  

#### Host networking

Firstly is that we are going to use the same network as the host.  We do this so that Pi-Hole will be receiving the DNS requests direct and not relayed via Docker.

{{< figure src="/images/unix/synology/host-networking.png" >}}

#### Port settings

As we are sharing the network with the host there are no port mapping requirements.

{{< figure src="/images/unix/synology/port-settings.png" >}}

#### Volumes

The latest image of Debian requires that the name servers configured has localhost first otherwise pihole fails to startup.  The official way to do this is to specify the `--dns 127.0.0.1` during the docker run startup however as we are using the Synology UI we are unable to do this.

My work around was to create my own file named `resolv.conf` and map this as a volume over the one inside the pihole container.

{{< figure src="/images/unix/synology/volume-settings.png" >}}

The file contains (using googles dns servers):

```
namserver 127.0.0.1
nameserver 8.8.8.8
nameserver 8.8.4.4
```
#### Environment

Lastly we configure 3 environment variables.

* `ServerIP` is the IP address of your NAS.
* `DNSMASQ_LISTENING` is set to `local` so that our DNS server will respond.
* `WEB_PORT` is set to any port that you would like the admin console on.  Values in the 8000 range are pretty good.

> An optional 4th can be `WEBPASSWORD` which allows you to set the password used in the UI.

{{< figure src="/images/unix/synology/environment.png" >}}

> `DNSMASQ_LISTENING` is required as the image runs `dnsmasq` listening to the `en0` interface which does not exist when using host networking on the Synology NAS.  Alternatively you can use the `INTERFACE` environment variable to be more specific.

### Configuring WebStation

Pi-Hole will redirect blocked DNS names to the IP of the Synology NAS.  For this to respond we need to install WebStation.  This will run on port 80 and will provide the blank areas where advert would have been seen.  

#### Apache

As we don’t want to manually modify any of the DSM files we need to run WebStation with the Apache web server instead of nginx.  Install Apache HTTP Server 2.4 using package manager.

> You can use nginx if you prefer but this would require modifying the nginx.conf file and the possibility of this being overwritten or causing damage.

In the WebStation application you should be able to see that Apache 2.4 is installed.

{{< figure src="/images/unix/synology/webstation-status.png" >}}

Next in the general settings make sure that we select Apache as the back-end server.

{{< figure src="/images/unix/synology/webstation-settings.png" >}}

#### WebStation files

WebStation will process requests on port 80 however most of these will not be valid paths that the Synology is expected (due to Pi-Hole mis-directing these requests) and therefore will respond with `404` file not found errors.  It’s ok but not ideal.  We can fix this (and this is the reason for Apache).

WebStation hosts files from `/volume1/web` so we need to create a new file named `.htaccess` (note the leading .) which contains the following.

```
ErrorDocument 404 /blocked-by-pihole.svg
ErrorDocument 500 /blocked-by-pihole.svg
```

This is now telling Apache that if you can’t find a file (will be most of the time due to Pi-Hole) then instead return the image `blocked-by-pihole.svg`.

And lastly place an image in the same place named `blocked-by-pihole.svg`.

> You can use whatever image you want but be sure to update the `.htaccess` correctly.

{{< figure src="/images/unix/synology/webstation-root.png" >}}

As requested in the comments below I have created a ZIP file containing the image and the `.htaccess` file which can be downloaded here:

[download it here](/downloads/pihole/web.zip)

Extract this into your `/volume1/web` folder.

### Conclusion

I hope you didn’t find this too daunting, it’s a few steps that makes Pi-Hole behave correctly on the Synology NAS running in a Docker container.  This correctly shows all client IP’s so I’m very happy.

As always, feel free to comment or ask for clarification.

Lastly, the image I used was downloaded from the [Pi-Hole Block Page Project](https://github.com/WaLLy3K/Pi-hole-Block-Page) 

{{< figure src="https://camo.githubusercontent.com/051550932e6840433baaa5d8091c432ef2546941/68747470733a2f2f77616c6c79336b2e6769746875622e696f2f7374796c652f626c6f636b65642e737667" >}}

> As of Pi-Hole 3.2 the separate Block Page project has been discontinued as custom pages have been rolled into Pi-Hole itself. We don’t need this project as WebStation is our block page server but we can still use the image.