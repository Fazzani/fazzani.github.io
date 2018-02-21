---
layout: default
title: Statefull docker swarm
categories: [docker, swarm, persistence]
category: docker
date:   2015-11-17 16:16:01 -0600
---
# Statefull docker swarm

- [Persistence Alternatives](https://opensource.ncsa.illinois.edu/confluence/display/NDS/Gluster+Alternatives+and+Cloud+Provider+Alternatives)
- All [docker plugins]

## Existing solutions

For quite a long time I have a pet project of a swarm cluster and the weak spot was definitely the distribution of the volumes. There are many volume plugins available and removing the plugins requiring a cloud storage or a complicated backend (i.e. to complex to install in small cluster) lead me to a small short list:

### Dist File systems types

- [GlusterFS]
- [NFS] : [install on swarm](https://attx-project.github.io/Shared-NFS-Swarm-Cloud.html), [Instal on swarm 2](http://collabnix.com/docker-1-12-swarm-mode-persistent-storage-using-nfs/)

### Storage management solutions

- [Rex-ray]
- [Infinit] : buyed by Docker (alpha version)
- [Flocker] : [flocker example install]
- [Convoy] powered by [Rancher] : NFS, EBS supported

## Using Cloudstor

 >create shared Cloudstor volumes using the docker volume create CLI:

```sh
docker volume create -d "cloudstor:aws" --opt backing=shared mysharedvol1
```

## Tips

> Trigger the failure on the active Swarm Node

```sh
docker node update --availability drain <machine>
```

### Rex-ray docker swarm

1. Install Rexray on each node and master

```sh
docker plugin install rexray/csi-nfs
```

>Let’s install REX-Ray on the 3 Docker Swarm nodes you have:

```sh
for each in $(docker-machine ls -q); do; docker-machine ssh $each "curl -sSL https://dl.bintray.com/emccode/rexray/install | sh -" ; done
```

2. Create volume

```sh
docker volume create -d rexray/csi-nfs -o host=151.80.235.155 -o export=/webgrab_volume webgrab
```

- docker compose volume config

```yaml
volumes:
  example:
    driver: local
    driver_opts:
      type: "nfs"
      o: "addr=$SOMEIP,nolock,soft,rw"
      device: ":$PathOnServer"
```

### NetShare

> Install netshare plugin on all hosts as per http://netshare.containx.io/docs/getting-started
[another source for install](https://attx-project.github.io/Shared-NFS-Swarm-Cloud.html)

```sh
wget https://github.com/ContainX/docker-volume-netshare/releases/download/v0.35/docker-volume-netshare_0.35_amd64.deb
sudo dpkg -i docker-volume-netshare_0.35_amd64.deb
```

> EDIT: One more thing, after installing the plugin you need to start it (sudo systemctl start docker-volume-netshare) then enable to start with the system (sudo systemctl enable docker-volume-netshare).

> Test netshare

```sh
docker run -i -t --volume-driver=nfs -v 18.194.42.216/mnt/test_volume:/data ubuntu /bin/bash
```

> add to autoload

```sh
sudo tee /etc/systemd/system/dockerefs.service <<-'EOF'
[Unit]
Description = Load docker plugin for efs
After = docker.service

[Service]
StartLimitInterval=5
StartLimitBurst=10
User=root
ExecStart = /usr/bin/docker-volume-netshare nfs
Restart=always
RestartSec=120
[Install]
WantedBy = multi-user.target
EOF
```

> Start the service

sudo systemctl start docker-volume-netshare.service

> Test verbose mount

```sh
sudo pkill -f docker-volume-netshare # kill any extant processes
cd /tmp
wget -O docker-volume-netshare https://github.com/ContainX/docker-volume-netshare/releases/download/v0.34/docker-volume-netshare_0.34_linux_amd64-bin
chmod a+x ./docker-volume-netshare
sudo ./docker-volume-netshare cifs --verbose=true
docker run -it --rm -v --volume-driver=nfs 18.194.42.216/tmp:/mount/tmp --name share_vol alpine ash

```

---
After discovering that this is massively undocumented,here's the correct way to mount a NFS volume using stack and docker compose.

The most important thing is that you need to be using version: "3.2" or higher. You will have strange and un-obvious errors if you don't.

The second issue is that volumes are not automatically updated when their definition changes. This can lead you down a rabbit hole of thinking that your changes aren't correct, when they just haven't been applied. Make sure you docker rm VOLUMENAME everywhere it could possibly be, as if the volume exists, it won't be validated.

The third issue is more of a NFS issue - The NFS folder will not be created on the server if it doesn't exist. This is just the way NFS works. You need to make sure it exists before you do anything.

(Don't remove 'soft' and 'nolock' unless you're sure you know what you're doing - this stops docker from freezing if your NFS server goes away)

---

[flocker  example install]: https://devops.profitbricks.com/tools/flocker/
[Flocker]:https://flocker-docs.clusterhq.com/en/latest/docker-integration/
[GlusterFS]:https://github.com/calavera/docker-volume-glusterfs
[Infinit]:https://devpost.com/software/infinit-docker-hackathon-1-12
[Convoy]:https://github.com/rancher/convoy
[Rancher]:https://rancher.com/
[NFS]:https://doc.ubuntu-fr.org/nfs
[Rex-ray]:https://rexray.readthedocs.io/en/stable/
[docker plugins]:https://docs.docker.com/engine/extend/legacy_plugins/#volume-plugins