
# Creating New Generic Docker machine

Remote : Add new User (dockeradmin)

```SHELL
useradd -m -d /home/dockeradmin -s /bin/bash dockeradmin
```
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