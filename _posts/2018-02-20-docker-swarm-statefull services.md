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

 create shared Cloudstor volumes using the docker volume create CLI:

```sh
docker volume create -d "cloudstor:aws" --opt backing=shared mysharedvol1
```

## Tips

- Trigger the failure on the active Swarm Node

```sh
docker node update --availability drain <machine>
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

[flocker  example install]: https://devops.profitbricks.com/tools/flocker/
[Flocker]:https://flocker-docs.clusterhq.com/en/latest/docker-integration/
[GlusterFS]:https://github.com/calavera/docker-volume-glusterfs
[Infinit]:https://devpost.com/software/infinit-docker-hackathon-1-12
[Convoy]:https://github.com/rancher/convoy
[Rancher]:https://rancher.com/
[NFS]:https://doc.ubuntu-fr.org/nfs
[Rex-ray]:https://rexray.readthedocs.io/en/stable/
[docker plugins]:https://docs.docker.com/engine/extend/legacy_plugins/#volume-plugins