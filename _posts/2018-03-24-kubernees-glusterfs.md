---
layout: default
title: Glusterfs kubernetes installation
categories: [GlusterFS, k8s, kubernetes, Heketi]
category: linux
date:   2018-03-24 16:16:01 -0600
---
# K8S - GlsterFS

## Laneling GlusterFs nodes

kubectl label node dkub-tst-ce7401 storagenode=glusterfs
kubectl label node dkub-tst-ce7402 storagenode=glusterfs

## Installing heketi-cli for dynamic provisioning

```sh
HEKETI_BIN="heketi-cli"      # heketi or heketi-cli
HEKETI_VERSION="6.0.0"       # latest heketi version => https://github.com/heketi/heketi/releases
HEKETI_OS="linux"            # linux or darwin

wget "https://github.com/heketi/heketi/releases/download/v${HEKETI_VERSION}/heketi-v${HEKETI_VERSION}.${HEKETI_OS}.amd64.tar.gz" -o "/tmp/heketi-v${HEKETI_VERSION}.${HEKETI_OS}.amd64.tar.gz" && \
tar xzvf /tmp/heketi-v${HEKETI_VERSION}.${HEKETI_OS}.amd64.tar.gz -C /tmp && \
rm -vf /tmp/heketi-v${HEKETI_VERSION}.${HEKETI_OS}.amd64.tar.gz && \
cp /tmp/heketi/${HEKETI_BIN} /usr/local/bin/${HEKETI_BIN}_${HEKETI_VERSION} && \
rm -vrf /tmp/heketi && \
cd /usr/local/bin && \
ln -vsnf ${HEKETI_BIN}_${HEKETI_VERSION} ${HEKETI_BIN} && cd

unset HEKETI_BIN HEKETI_VERSION HEKETI_OS
```

## Tips

### Glustfs commands

* gluster help
* gluster pool list
* gluster peer status
* gluster volume info

### Purge the glusterd directory

Run these commands on all cluster nodes:

```sh
service glusterd stop
mv /var/lib/glusterd/glusterd.info /tmp/.
rm -rf /var/lib/glusterd/*
mv /tmp/glusterd.info /var/lib/glusterd/.
service glusterd start
```

## References

* [Gluster install Source valid](https://github.com/kubernetes/examples/tree/master/staging/volumes/glusterfs)
* [Gluster install heketi to test](https://techdev.io/en/developer-blog/deploying-glusterfs-in-your-bare-metal-kubernetes-cluster)
* [Heketi Source](https://github.com/psyhomb/heketi)
* [Understanding Gluster architecture && How Install it on CentOS 7](https://www.slothparadise.com/how-to-install-glusterfs-on-centos-7/)
