---
title: Using Kubernetes ExternalDNS & MetalLB with a Home/Bare Metal K8S
date: 2018-07-25 00:00:00 -0700

---
I run a Kubernetes cluster at home because.....nerd reasons.   I love to have a place to experiment with new things outside of my generously provided work labs (GCE and vSphere based).

I've recently been playing with ways to expose services on common ports, with virtual IPs (VIPs) and with reasonable DNS entries.  I finally got it working the way I always hoped today, so I thought I'd share the process.

The first step is that we need a tool to assign an 'external' IP to services that are created.  Now, most services services created in Kubernetes default to the ClusterIP type, where only a in-cluster IP is assigned to the service.   I need something outside the cluster so the machines on the rest of my network can use that.   That type of service is called a 'LoadBalancer' service, and works out of the box with GCP, AWS and a couple major firewall hardware vendors like F5.   I needed something open source, and entirely software.

Enter MetalLB, an implementation of the LoadBalancer spec for bare metal clusters (or really any cluster that isn't in AWS or GCP).  As the MetalLB website suggests:

> Bare metal cluster operators are left with two lesser tools to bring user traffic into their clusters, “NodePort” and “externalIPs” services. Both of these options have significant downsides for production use, which makes bare metal clusters second class citizens in the Kubernetes ecosystem.

MetalLB fixes that:

> MetalLB aims to redress this imbalance by offering a Network LB implementation that integrates with standard network equipment, so that external services on bare metal clusters also “just work” as much as possible.

Very cool!   MetalLB has a couple modes...one of them is more 'enterprisey' and focuses on using BGP connections to upstream routers to do some clever route advertisements and achieves some really cool effects.   However, it also has a second mode wherein it simply uses gratuitous L2 ARP requests to 'advertise' itself as willing to accept traffic on a given IP address.   This works GREAT in homelabs.

So lets start to configure that!

The MetalLB helm chart works great out of the box:

    helm install --name metallb stable/metallb

It adds quite a few objects, including namespace, ServiceAccounts, ClusterRoles, RoleBindings, DaemonSets (for the 'speakers' that advertise their relevant VIPs), and a Deployment for its controller.  I've looked through, and the permissions all seem pretty reasonable, and there's even handy annnotations to automatically enable tools like Prometheus to gather the data.  

After all is said and done, you get a full set of everything:

    ╰ kubectl get all --all-namespaces | grep metallb
    metallb-system   pod/controller-997f5bbb7-rc9kw             1/1       Running   0          4d
    metallb-system   pod/speaker-262wn                          1/1       Running   0          4d
    metallb-system   pod/speaker-7gbwr                          1/1       Running   0          4d
    metallb-system   pod/speaker-dgwq4                          1/1       Running   0          4d
    metallb-system   daemonset.apps/speaker           3         3         3         3            3           <none>                          4d
    metallb-system   deployment.apps/controller             1         1         1            1           4d
    metallb-system   replicaset.apps/controller-997f5bbb7             1         1         1         4d

You'll then need to define a ConfigMap containing the config data (like your IP pool) for the LoadBalancer to use:

    apiVersion: v1
    kind: ConfigMap
    metadata:
      namespace: metallb-system
      name: config
    data:
      config: |
        address-pools:
        - name: default
          protocol: layer2
          addresses:
          - 192.168.0.230-192.168.0.250

Now we can deploy a service as a 'LoadBalancer' type rather than a ClusterIP:

First we can deploy an nginx deployment:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-nginx
    spec:
      selector:
        matchLabels:
          run: my-nginx
      replicas: 1
      template:
        metadata:
          labels:
            run: my-nginx
        spec:
          containers:
          - name: my-nginx
            image: nginx
            ports:
            - containerPort: 80

Then we can expose a service:

    apiVersion: v1
    kind: Service
    metadata:
      name: my-nginx
      labels:
        run: my-nginx
    spec:
      type: LoadBalancer
      ports:
      - port: 80
        protocol: TCP
      selector:
        run: my-nginx
    

**_Notice the 'type: LoadBalancer' option_**

And now check it out, not only do I have a cluster IP, but MetalLB has assigned an IP address from the configuration range 