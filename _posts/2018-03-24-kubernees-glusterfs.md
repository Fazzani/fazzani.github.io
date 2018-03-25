---
title: Glusterfs kubernetes installation
categories: [GlusterFS, k8s, kubernetes, Heketi]
category: linux
last_modified_at: 2018-03-24 16:16:01 -0600
---
# K8S - GlsterFS

kubectl exec -it exampleapp-649bf696cc-kvmcz -- /bin/bash

1. Creating PVC for exampleapp (nginx-pvc-gluster.yml)

```yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: gluster-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 2Gi
```

2. Creating nginx service (nginx-svc-gluster.yml)

```yml
apiVersion: v1
kind: Service
metadata:
  name: exampleapp-svc
  namespace: default
spec:
  type: NodePort
  selector:
    app: exampleapp
  ports:
  - port: 8080
    targetPort: 80
    protocol: TCP
    name: http
  - port: 443
    protocol: TCP
    name: https
```

3. Creating nginx deployement (nginx-dep-gluster.yml)

```yml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: exampleapp
  labels:
    app: exampleapp
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: exampleapp
    spec:
     containers:
       - name: nginx-pod1
         image: gcr.io/google_containers/nginx-slim:0.8
         ports:
         - name: web
           containerPort: 80
         volumeMounts:
         - name: gluster-vol1
           mountPath: /usr/share/nginx/html
     volumes:
     - name: gluster-vol1
       persistentVolumeClaim:
         claimName: gluster-pvc
```

4. Creating ingress for exampleapp (nginx-ingress-gluster.yml)

```yml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: exampleapp-ingress
  namespace: default
  annotations:
    kubernetes.io/tls-acme: "true"
    kubernetes.io/ingress.class: "nginx"
spec:
  tls:
  - hosts:
    - nginx.k8s.seloger.tools
    secretName: exampleapp-tls
  rules:
  - host: nginx.k8s.seloger.tools
    http:
      paths:
      - path: /
        backend:
          serviceName: exampleapp-svc
          servicePort: 80
```

```sh
# Test Peristence
kubectl exec -it exampleapp-649bf696cc-kvmcz -- /bin/bash
kubectl exec -it exampleapp-649bf696cc-kvmcz -- bash -c "echo '<h1>Hello at `date +%H:%M:%S`</h1>' > /usr/share/nginx/html/index.html"
```

## Labeling GlusterFs nodes

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

---

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

---

## References

* [Gluster install Source valid](https://github.com/kubernetes/examples/tree/master/staging/volumes/glusterfs)
* [Gluster install heketi to test](https://techdev.io/en/developer-blog/deploying-glusterfs-in-your-bare-metal-kubernetes-cluster)
* [Heketi Source](https://github.com/psyhomb/heketi)
* [Understanding Gluster architecture && How Install it on CentOS 7](https://www.slothparadise.com/how-to-install-glusterfs-on-centos-7/)
