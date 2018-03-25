---
title: Docker machine for remote access (Generic)
categories: docker
category: docker
last_modified_at: 2015-11-17 16:16:01 -0600
tags:
  - docker
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

ssh-copy-id -i <cert_file_name.pub> dockeradmin@remote-server
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

docker-machine --debug create -d generic \
--generic-ssh-user dockeradmin \
--generic-ssh-key aws.pem \
--generic-ssh-port 22 \
--label size=small \
--label provider=ovh \
--engine-label label=debian-x86_64-8g \
--engine-label arch=x86_64 \
--engine-label mem=8 \
--engine-storage-driver devicemapper \
--generic-ip-address 18.194.42.216 \
synker.machine.aws

```

## Various

Switch User

```SHELL
sudo -u dockeradmin -s
```

Check docker deamon port

```SHELL
sudo netstat -tunlp | grep docker
```

** To copy all from Local Location to Remote Location (Upload) **

```shell
scp -r /path/from/destination username@hostname:/path/to/destination
```

** To copy all from Remote Location to Local Location (Download) **

```shell
scp -r username@hostname:/path/from/destination /path/to/destination
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
