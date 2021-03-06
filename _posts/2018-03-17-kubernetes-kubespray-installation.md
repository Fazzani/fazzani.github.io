---
title: Kubespray installation
categories: [kubernetes, kubespray, cluster, ansible]
category: kubernetes
last_modified_at: 2018-03-17 16:16:01 -0600
tags:
  - kubespray
  - kubernetes
  - ansible
---
# Clusturing Kubernetes with Kubespray

## Installation procedure

```sh
# Install requirements on local machine
yum -y install https://centos7.iuscommunity.org/ius-release.rpm
yum -y install python36u
pip install ansible netaddr

# Clone Kubespray
git clone [https://github.com/kubernetes-incubator/kubespray.git](https://github.com/kubernetes-incubator/kubespray.git)
cd [kubespray](https://github.com/kubernetes-incubator/kubespray.git)
git checkout -b tags/v2.4.0

# Declare all nodes and generate ansible inventory file
declare -a IPS=(10.2.1.4 10.2.1.5 10.2.1.6 10.2.1.7 10.2.1.8)
CONFIG_FILE=inventory/mycluster/hosts.ini python3.6 contrib/inventory_builder/inventory.py ${IPS[@]}
cat inventory/mycluster/hosts.ini

# Copy ssh key to all nodes
for IP in ${IPS[@]}; do ssh-copy-id root@$IP; done

# Adjust cluster creation variables
nano inventory/mycluster/group_vars/all.yml
nano inventory/mycluster/group_vars/k8s-cluster.yml

# Launching ansible playbook to create cluster
ansible-playbook -i inventory/mycluster/hosts.ini cluster.yml -b -v -T 60 \
--private-key=~/.ssh/id_rsa -e kube_api_passwd=CentOS2018 -e dashboard_enabled=true --flush-cache
```

## Dashboard access

[Dashboard login page](https://10.2.1.5:6443/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/login)

```sh
# Get auth tokens
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep kube_user | awk '{print $1}')
# Get default user password ( only when the basic is activated)
cat inventory/mycluster/credentials/kube_user
```

## Using kubectl remotely (from installation computer)

> From kubespray folder execute the command below (the artifact generation must be enabled)

```sh
./artifacts/kubectl cluster-info  --kubeconfig ./artifacts/admin.conf
```

## Verify the cluster state

```sh
# ping all hosts
ansible -i inventory/mycluster/hosts.ini  --private-key=~/.ssh/id_rsa -m ping -vvvv all
# get all pods
kubectl get pods -o wide --all-namespaces
# Check services within your cluster
kubectl get svc
```

## kubernetes-dashboard service access

```sh
kubectl describe svc kubernetes-dashboard --namespace=kube-system
kubectl describe services kubernetes-dashboard --namespace=kube-system | grep NodePort
```

## Reset and remove kubernetes

> Reset Host

```sh
kubeadm reset && \
yum remove -y kubeadm kubectl kubelet kubernetes-cni kube*
```

> Reset cluster with kubespray

```sh
ansible-playbook -i inventory/mycluster/hosts.ini reset.yml -b -v \
--private-key=~/.ssh/id_rsa
```

## Scaling cluster

[dynamic inventory](https://docs.ansible.com/ansible/intro_dynamic_inventory.html)

> Adding nodes

```sh
ansible-playbook -i inventory/mycluster/hosts.ini scale.yml -b -v \
  --private-key=~/.ssh/private_key
```

> Remove nodes

```sh
ansible-playbook -i inventory/mycluster/hosts.ini remove-node.yml -b -v \
  --private-key=~/.ssh/private_key
```

### Create a admin dashboard token

```sh

cat <<'EOF' > dashboard-admin.yaml 
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard
  labels:
    k8s-app: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard
  namespace: kube-system

EOF

kubectl create -f dashboard-admin.yaml 
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep kubernetes-dashboard | awk '{print $1}')
```

### kubectl configure context access

```sh
mkdir ~/.kube && cd ~/.kube && touch config

kubectl config --kubeconfig=config set-cluster multi-master2 --server=https://x.x.x.x:6443 --insecure-skip-tls-verify
kubectl config --kubeconfig=config set-credentials user_master2 --token=eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC10b2tlbi1sdDl4ZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImYxMzFlMWM5LTMwMWMtMTFlOC1iN2QwLTAwNTA1NmFlNDJjMyIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTprdWJlcm5ldGVzLWRhc2hib2FyZCJ9.gBXjle_oGSxRKUBmJ7_aJUUykBDipj4YezSGe5zBFwergpL25jPPyGmgTnzBeP3N5FzHxA2rkFZuxslQ04yccxEg6h3wKztQD8hXMOgPmwj330vUP0-_obLyi83G3Ta7QmlY23-Ml2ely2mvZsW_hZNg1sMCjKCi9Uu3s5PEZkdD9dCqRTsD_3j-tIpifjXQT29Gk-c0ZlPQyF8V1m--E2q2-cqmpXc_g-vsQBw7nYrm0iEfnQCB7aG5f2NiGH-Urx3uJSsGOFP_HBYmzXpWXghuH_G7Edl_HrUdsk8qA_af702J8XiYyhOdrA3Kr7epMgFMYh1Dsd2ck5GRLsEBdg
kubectl config --kubeconfig=config set-context multi-master-ctx2 --cluster=multi-master2 --namespace=default --user=user_master2
kubectl config --kubeconfig=config use-context multi-master-ctx2
```

## Comparisons

### Kubespray vs [Kops](https://github.com/kubernetes/kops)

>Kubespray runs on bare metal and most clouds, using Ansible as its substrate for
provisioning and orchestration. Kops performs the provisioning and orchestration
itself, and as such is less flexible in deployment platforms. For people with
familiarity with Ansible, existing Ansible deployments or the desire to run a
Kubernetes cluster across multiple platforms, Kubespray is a good choice. Kops,
however, is more tightly integrated with the unique features of the clouds it
supports so it could be a better choice if you know that you will only be using
one platform for the foreseeable future.

### Kubespray vs [Kubeadm](https://github.com/kubernetes/kubeadm)

>Kubeadm provides domain Knowledge of Kubernetes clusters' life cycle
management, including self-hosted layouts, dynamic discovery services and so
on. Had it belonged to the new [operators world](https://coreos.com/blog/introducing-operators.html),
it may have been named a "Kubernetes cluster operator". Kubespray however,
does generic configuration management tasks from the "OS operators" ansible
world, plus some initial K8s clustering (with networking plugins included) and
control plane bootstrapping. Kubespray [strives](https://github.com/kubernetes-incubator/kubespray/issues/553)
to adopt kubeadm as a tool in order to consume life cycle management domain
knowledge from it and offload generic OS configuration things from it, which
hopefully benefits both sides.

## References

- [Doc officielle](https://github.com/kubernetes-incubator/kubespray)
- [Kubspray install 1](https://dickingwithdocker.com/2017/08/deploying-kubernetes-vms-kubespray)
- [Kubspray install 2](https://linode.com/docs/applications/containers/deploy-minio-on-kubernetes-using-kubespray-and-ansible)
- [Kubspray install 3](https://blog.zwindler.fr/2017/12/05/installer-kubernetes-kubespray-ansible/)
- [Ingress with nginx](https://medium.com/@olegsmetanin/how-to-setup-baremetal-kubernetes-cluster-with-kubespray-and-deploy-ingress-controller-with-170cdb5ac50d)
- [Creating dashboad user](https://github.com/kubernetes/dashboard/wiki/Creating-sample-user)
- [kargo  cli example](https://asciinema.org/a/065mhh5pzmxcwxgp6evebarvd?speed=4)
- [vSphere](https://github.com/kubernetes-incubator/kubespray/blob/master/docs/vsphere.md)
- [Awesome links](https://ramitsurana.github.io/awesome-kubernetes/)
- [Sample app can easly be upgraded](https://github.com/honestbee/hello-drone-helm)

[helm]:https://docs.helm.sh
