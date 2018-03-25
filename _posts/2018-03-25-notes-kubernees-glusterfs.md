---
layout: default
title: Notes & Tips => Glusterfs kubernetes Heketi
categories: [GlusterFS, k8s, kubernetes, Heketi]
category: linux
date: 2018-03-25 18:16:01 0100
---
# Notes

## Install Heketi for GlusterFS

```sh
mkdir -p /data/heketi/{db,.ssh} && chmod 700 /data/heketi/.ssh
ssh-keygen -t rsa -b 2048 -f /data/heketi/.ssh/id_rsa
for NODE in dkub-tst-ce7401 dkub-tst-ce7402; do scp -r /data/heketi/.ssh root@${NODE}:/data/heketi; done

for NODE in dkub-tst-ce7401 dkub-tst-ce7402; do cat /data/heketi/.ssh/id_rsa.pub | ssh root@${NODE} "cat >> /root/.ssh/authorized_keys"; done

HEREKI_SERVER_URL="http://dkub-tst-ce7402:31251"
curl -s $HEREKI_SERVER_URL/hello
heketi-cli --user admin --secret password --server $HEREKI_SERVER_URL cluster list
heketi-cli --user admin --secret password --server $HEREKI_SERVER_URL topology info

heketi-cli --user admin --secret password --server $HEREKI_SERVER_URL topology load --json topology.json
```

## Config kubectl

```sh
kubectl config --kubeconfig=config set-cluster multi-master2 --server=https://10.2.1.118:6443 --insecure-skip-tls-verify
kubectl config --kubeconfig=config set-context multi-master-ctx2 --cluster=multi-master2 --namespace=default
kubectl config --kubeconfig=config use-context multi-master-ctx2
kubectl config --kubeconfig=config set-credentials user_master2 --token=`token`
```

---

## Tips

### Running commands on ansible inventory

```sh
ansible kube-node -a "wipefs -af /dev/sdb1" -u root -i inventory/seloger-glusterfs/hosts.ini
```

### Labeling k8s node

```sh
kubectl label node ddev-dck-ce7401 storagenode=glusterfs
kubectl label node dkub-tst-ce7401 storagenode=glusterfs
```

sudo mount -t glusterfs node2:webdir  /var/www/

### Erase partition

xfs_repair -L /dev/sdb
xfs_repair: /dev/sdb contains a mounted filesystem
>Solution

* edit /etc/fstab file to comment /data1 entry.
* reboot server
* run 'xfs_repair /dev/sdb' again, it succeeded.
* edit /etc/fstab file to uncomment /data1 entry.
* run 'mount -a' to mount /data1 entry

### Purge the glusterd directory

Run these commands on all cluster nodes:

* service glusterd stop
* mv /var/lib/glusterd/glusterd.info /tmp/.
* rm -rf /var/lib/glusterd/*
* mv /tmp/glusterd.info /var/lib/glusterd/.
* service glusterd start

---

## References

* [Gluster install Source valid](https://github.com/kubernetes/examples/tree/master/staging/volumes/glusterfs)
* [Gluster install heketi to test](https://techdev.io/en/developer-blog/deploying-glusterfs-in-your-bare-metal-kubernetes-cluster)
* [Source Heketi](https://github.com/psyhomb/heketi)