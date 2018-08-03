---
title: Kubernetes NFS on Synology
date: 2018-08-03 00:00:00 -0700

---
In my ongoing quest to pretend that my Kubernetes homelab resembles a real datacenter, I've set up [automated DNS](https://blog.cowger.us/2018/07/26/using-kubernetes-externaldns-metallb-with-a-home-bare-metal-k8s-part-2.html) and even a [software LoadBalancer](https://blog.cowger.us/2018/07/25/using-kubernetes-externaldns-with-a-home-bare-metal-k8s.html).

The next step is to enable external storage for persistent applications.  There's a ton of options for this, including [Rook](https://rook.io/), [ScaleIO/VxFlexOS](https://github.com/thecodeteam/csi-scaleio), and even a [self-hosted NFS setup](https://github.com/kubernetes-incubator/external-storage/tree/master/nfs).   But I already have a very nice SOHO storage system, a [Synology DS918+](https://www.synology.com/en-us/products/DS918+) with 8TB of space and a 1TB read/write cache....I have no need for something else.

What I wanted to do was to use my Synology NFS server as a persistent store.   Turns out there is a [nfs-client provisioner](https://github.com/kubernetes-incubator/external-storage/tree/master/nfs-client) that simply creates folders on an existing NFS shares and mounts them into containers.  Its not the most secure thing out there, but it will work for my home lab.

First, I needed to create a share that I would use, and enable the proper options on the Synology:

![](/uploads/Screen Shot 2018-08-01 at 2.59.56 PM.png)

![](/uploads/Screen Shot 2018-08-01 at 3.00.05 PM.png)

The hardest challenge I faced was figuring out user mapping - the workers need to be able to mount the NFS store as `root`, but it wasn't obvious to me that the 'No Mapping' option for 'Squash' was equivalent to `no root squash` under traditional unix.

From there it was a matter of installing the provisioner.

First, deploy the provisioner agent itself:

```yaml
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: nfs-client-provisioner
spec:
  replicas: 1
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          image: quay.io/external_storage/nfs-client-provisioner:latest
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: exaforge.com/whale
            - name: NFS_SERVER
              value: 192.168.0.10
            - name: NFS_PATH
              value: /volume1/kubernetes
      volumes:
        - name: nfs-client-root
          nfs:
            server: 192.168.0.10
            path: /volume1/kubernetes
```

Then, create the storage class, adding the default class annotation:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
  name: managed-nfs-storage
provisioner: exaforge.com/whale
reclaimPolicy: Delete
volumeBindingMode: Immediate
```

From there, its just a matter of deploying a PVC or some Pod invilving a persistent volume.   Happy provisioning!