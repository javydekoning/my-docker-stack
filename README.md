# Docker Stack for my home-lab

This repository is used to keep my home-lab running. I run a [low-power x86 machine](https://www.asrock.com/mb/Intel/N3150DC-ITX/) with Alpine Linux.

When I got this machine in 2016 I wanted to keep the configuration as simple to maintain as possible. Some reasons for choosing Docker-Compose my homelab:

1. Auto-restart of containers. If the process dies, a new container will be created.
2. DNS management.
   1. I want to run PiHole to remove adds from all devices on my network (e.g. smart TV, tables etc).
   2. I also want to use DNSMASQ to intercept and redirect queries to my containers.
3. Hardware replacement made easy. Just clone my repo, run `docker-compose up` and we are done!
4. Easy to keep updated. All applications are running in a container and I'm using [WatchTower](https://github.com/containrrr/watchtower) to keep my containers updated.

## Known issues

NFS discovery will not work! Point Kodi to the absolute path: `nfs://<dockerhost ip>/nfs` (Server Address: <dockerhost ip>, Remote Path: nfs)

## Prepping the Alpine Docker host

First we install the server from **vanilla** Alpine ISO (USB). Vanilla is required if you directly on your hardware.

```Bash
setup-alpine
us
us-intl
dhcp
eth0
```

### setup ssh keys

Enable ssh login using password, copy keys, disable password login again.

```Bash
vi /etc/ssh/sshd_config
/etc/init.d/sshd restart
ssh-copy-id root@127.0.0.1
```

### add repo

Add the following repo urls to `/etc/apk/repositories`. These are not version pinned and allow you to keep upgrading Alpine to the latest version.

```Bash
http://dl-cdn.alpinelinux.org/alpine/latest-stable/main/
http://dl-cdn.alpinelinux.org/alpine/latest-stable/main/community
```

### Install dependencies

Install dependencies required for `docker-compose`

```Bash
apk update
apk add docker py-pip vim git curl python-dev libffi-dev openssl-dev gcc libc-dev  make
rm -rf /etc/timezone
echo "Europe/Amsterdam" >> /etc/timezone
```

### install docker & docker compose

```Bash
rc-update add docker boot
service docker start
pip install docker-compose
```

### prep docker for Prometheus

```Bash
vi /etc/docker/daemon.json
```

#### /etc/docker/daemon.json:

```json
{
  "metrics-addr" : "0.0.0.0:9323",
  "experimental" : true
}
```

### Clone repo

```Bash
git clone git@github.com:javydekoning/my-docker-stack.git /srv/docker
mkdir /srv/docker/downloads && chmod -R a+rw /srv/docker/downloads/
```

### add my data disk

Edit `/etc/fstab/` add this:

```Bash
/dev/sdb1       /srv/docker/downloads    ext4    defaults    0    1
```

Next mount disk:

```Bash
mount -a
chmod -R a+rw /srv/docker/prometheus/
````