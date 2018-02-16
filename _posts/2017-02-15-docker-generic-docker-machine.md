---
layout: default
title: Docker machine for remote access (Generic)
categories: docker
category: docker
date:   2015-11-17 16:16:01 -0600
---
# Creating New Generic Docker machine

Remote : Add new User (dockeradmin)

```SHELL
useradd -m -d /home/dockeradmin -s /bin/bash dockeradmin
passwd dockeradmin
```

Local : Allow Sudo Access for dockeradmin

```SHELL
 echo "dockeradmin     ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
```

Local : Generate SSH Public-Private Key Pair on Local Host

```SHELL
ssh-keygen

ssh-copy-id dockeradmin@remote-server
```

Note: During ssh-keygen, don’t give any passphrase. Leave it empty.

Remote : puting cert to authorized_keys of the new user (dockeradmin)

```SHELL
cat aws.pem.pub > /home/dockeradmin/.ssh/authorized_keys
```

Local : TEST SSH connection

```SHELL
ssh -i aws.pem dockeradmin@18.194.42.216
```

Local : Create docker-machine

```SHELL

docker-machine create -d generic \
--generic-ssh-user dockeradmin \
--generic-ssh-key aws.pem \
--generic-ssh-port 22 \
--engine-storage-driver devicemapper \
--generic-ip-address 18.194.42.216 \
synker.machine.aws

```

### Various

```SHELL

wget "https://drive.google.com/uc?export=download&id=1Y3K3G-n5IUVcM7vMMbAm8_OHFkJu80sT" -o aws.pem

ssh-keygen -y -f aws.pem > aws.pem.pub

```

Switch User

```SHELL
sudo -u dockeradmin -s
```

Check docker deamon port

```SHELL
sudo netstat -tunlp | grep docker
```

### For AWS

add appropriate SecurityGroups TCP connections input and output

## Docker Swarm

```SHELL
docker-machine ssh <master>
```

- Init swarm on master node

```SHELL
docker swarm init && exit
```

- Joining worker to master

```SHELL
docker swarm join \
    --token <SWMTKN-1-5so41imzseot1fsldk97k5nh9zhdmj11dio8s2b7r96a2966oq-87uhvtps1jgje87bfbhrlv712> \
    <public_worker_ip : 192.168.99.100>:2377
```

- Swarm nodes listing

```SHELL
 docker node ls
```