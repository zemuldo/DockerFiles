# DockerFiles

  Getting started with Docker on Arch Linux

   Installation

```
sudo pacman -S docker
```
Docker Commandline reference is available here
[Docker Commandline Reference](https://docs.docker.com/edge/engine/reference/commandline/docker/])
To see if docker is installed Run
Note that this command may require root privilages

```
docker info
```

Configuration
Storage driver
The docker storage driver (or graph driver) has huge impact on performance. Its job is to store layers of container images efficiently, that is when several images share a layer, only one layer uses disk space. The compatible option, `devicemapper` offers suboptimal performance, which is outright terrible on rotating disks. Additionally, `devicemappper` is not recommended in production.

As Arch linux ships new kernels, there is no point using the compatibility option. A good, modern choice is overlay2.

To see current storage driver, run # docker info | head, modern docker installation should already use overlay2 by default.

To set your own choice of storage driver, create a Drop-in snippet and use -s option to dockerd (use systemctl edit docker):

```
/etc/systemd/system/docker.service.d/override.conf
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// -s overlay2
```
Note that the ExecStart= line is needed to drop inherited ExecStart.

Further information on options is available on the user guide.

Remote API
To open the Remote API to port 4243 manually, run:

```
# /usr/bin/dockerd -H tcp://0.0.0.0:4243 -H unix:///var/run/docker.sock
-H tcp://0.0.0.0:4243 part is for opening the Remote API.

-H unix:///var/run/docker.sock part for host machine access via terminal.
```
Remote API with systemd
To start the remote API with the docker daemon, create a Drop-in snippet with the following content:

```
/etc/systemd/system/docker.service.d/override.conf
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:4243 -H unix:///var/run/docker.sock
```
Daemon socket configuration
The docker daemon listens to a Unix socket by default. To listen on a specified port instead, create a Drop-in snippet with the following content:

```
/etc/systemd/system/docker.socket.d/socket.conf
[Socket]
ListenStream=0.0.0.0:2375
```
Proxies
Proxy configuration is broken down into two. First is the host configuration of the Docker daemon, second is the configuration required for your container to see your proxy.

Proxy configuration
Create a Drop-in snippet with the following content:

```
/etc/systemd/system/docker.service.d/proxy.conf
[Service]
Environment="HTTP_PROXY=192.168.1.1:8080"
Environment="HTTPS_PROXY=192.168.1.1:8080"
```
Note: This assumes 192.168.1.1 is your proxy server, do not use 127.0.0.1.
Verify that the configuration has been loaded:

```
# systemctl show docker --property Environment
Environment=HTTP_PROXY=192.168.1.1:8080 HTTPS_PROXY=192.168.1.1:8080
```
Container configuration
The settings in the docker.service file will not translate into containers. To achieve this you must set ENV variables in your Dockerfile thus:

```
 FROM base/archlinux
 ENV http_proxy="http://192.168.1.1:3128"
 ENV https_proxy="https://192.168.1.1:3128"
 ```
Docker provide detailed information on configuration via ENV within a Dockerfile.

Configuring DNS
By default, docker will make resolv.conf in the container match /etc/resolv.conf on the host machine, filtering out local addresses (e.g. 127.0.0.1). If this yields an empty file, then Google DNS servers are used. If you are using a service like dnsmasq to provide name resolution, you may need to add an entry to the /etc/resolv.conf for docker's network interface so that it is not filtered out.

Running Docker with a manually-defined network
If you manually configure your network using systemd-network version 220 or higher, containers you start with Docker may be unable to access your network. Beginning with version 220, the forwarding setting for a given network (net.ipv4.conf.<interface>.forwarding) defaults to off. This setting prevents IP forwarding. It also conflicts with Docker which enables the net.ipv4.conf.all.forwarding setting within a container.

To work around this, edit the <interface>.network file in /etc/systemd/network/ on your Docker host add the following block:

```
/etc/systemd/network/<interface>.network
[Network]
...
IPForward=kernel
...
```

This configuration allows IP forwarding from the container as expected.

Images location
By default, docker images are located at /var/lib/docker. They can be moved to other partitions. First, stop the docker.service.

If you have run the docker images, you need to make sure the images are unmounted totally. Once that is completed, you may move the images from /var/lib/docker to the target destination.

Then add a Drop-in snippet for the docker.service, adding the --data-root parameter to the ExecStart:

```
/etc/systemd/system/docker.service.d/docker-storage.conf
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd --data-root=/path/to/new/location/docker -H fd://
Insecure registries
If you decide to use a self signed certificate for your private registry, Docker will refuse to use it until you declare that you trust it. Add a Drop-in snippet for the docker.service, adding the --insecure-registry parameter to the dockerd:

/etc/systemd/system/docker.service.d/override.conf
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// --insecure-registry my.registry.name:5000

```
Images
Arch Linux
The following command pulls the archlinux/base x86_64 image.

```
# docker pull archlinux/base
```
See also README.md.

Debian
The following command pulls the debian x86_64 image.

```
# docker pull debian
```
Manually
Build Debian image with debootstrap:

```
# mkdir jessie-chroot
# debootstrap jessie ./jessie-chroot http://http.debian.net/debian/
# cd jessie-chroot
# tar cpf - . | docker import - debian
# docker run -t -i --rm debian /bin/bash
```

Arch Linux image with snapshot repository
Arch Linux on Docker can become problematic when multiple images are created and updated each having different package versions. To keep Docker containers with consistent package versions, an unofficial Docker image with a snapshot repository is available. This allows installing new packages from the official repository as it was on the day that the snapshot was created.

```
$ docker pull pritunl/archlinux:latest
$ docker run --rm -t -i pritunl/archlinux:latest /bin/bash
```

Alternatively, you could use Arch Linux Archive by freezing /etc/pacman.d/mirrorlist

```
 Server=https://archive.archlinux.org/repos/2020/01/02/$repo/os/$arch
 ```

Clean Remove Docker + Images
In case you want to remove Docker entirely you can do this by following the steps below:

Note: Do not just copy paste those commands without making sure you know what you are doing.
Check for running containers:

```
# docker ps
```

List all containers running on the host for deletion:

```
# docker ps -a
```
Stop a running container:

```
# docker stop <CONTAINER ID>
```

Killing still running containers:

```
# docker kill <CONTAINER ID>
```
Delete all containers listed by ID:

```
# docker rm <CONTAINER ID>
```
List all Docker images:

```
# docker images
```
Delete all images by ID:

```
# docker rmi <IMAGE ID>
```
Delete all Docker data (purge directory):

```
# rm -R /var/lib/docker
```
Useful tips
To grab the IP address of a running container:

```
$ docker inspect --format '{{ .NetworkSettings.IPAddress }}' <container-name OR id>
172.17.0.37
```
Troubleshooting
Cannot start a container with systemd 232
Append systemd.legacy_systemd_cgroup_controller=yes as kernel parameter, see bug report for details.

Deleting Docker Images in a BTRFS Filesystem
Deleting docker images in a btrfs filesystem leaves the images in /var/lib/docker/btrfs/subvolumes/ with a size of 0. When you try to delete this you get a permission error.

```
 # docker rm bab4ff309870
 # rm -Rf /var/lib/docker/btrfs/subvolumes/*
 rm: cannot remove '/var/lib/docker/btrfs/subvolumes/85122f1472a76b7519ed0095637d8501f1d456787be1a87f2e9e02792c4200ab': Operation not permitted
```
This is caused by btrfs which created subvolumes for the docker images. So the correct command to delete them is:

```
 # btrfs subvolume delete /var/lib/docker/btrfs/subvolumes/85122f1472a76b7519ed0095637d8501f1d456787be1a87f2e9e02792c4200ab
docker0 Bridge gets no IP / no internet access in containers
```
Docker enables IP forwarding by itself, but by default systemd overrides the respective sysctl setting. The following disables this override (for all interfaces):

```
# cat > /etc/systemd/network/ipforward.network <<EOF
[Network]
IPForward=kernel
EOF

# cat > /etc/sysctl.d/99-docker.conf <<EOF
net.ipv4.ip_forward = 1
EOF
```

```
# sysctl -w net.ipv4.ip_forward=1
```