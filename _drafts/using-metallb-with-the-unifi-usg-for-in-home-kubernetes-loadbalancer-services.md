---
title: Using MetalLB with the Unifi USG for in-home Kubernetes LoadBalancer Services
date: 2019-02-10 00:00:00 -0800

---
In a recent post, I detailed how to use Layer2 advertisements with MetalLB to simulate internal LoadBalancers for Kubernetes.

However, I have a nice Ubiqitui USG Router that does all sorts of nice stuff like iBGP, and I wanted to use that to be able to advertise an entirely different address space exclusively for the use of k8s LoadBalancer Services.   MetalLB supports BGP sessions, so I gave it a try setting it up - and I was successful.  Here's how I did it:

First, we need to configure the USG to speak BGP internally - its fairly simple, but does require use of the CLI, as BGP configuration is not yet in the Unifi Controller GUI:

1. Enter configuration mode with `configure`
2. Configure the router for BGP using its own IP address as the router id.  We'll also use the 'private' AS (autonomous system) number of 64512.  Think of this like the RFC1918 address space (like 192.168.0.0/24).  In my case, my internal network is 10.0.0.0/24:  
   `set protocols bgp 64512 parameters router-id 10.0.0.1`
3. Next, configure each worker on the network as a possible peer.   Note that if you add or remove workers from your k8s cluster, you should update this.   Again, we'll be using the private AS:  
   `set protocols bgp 64512 neighbor 10.0.0.24 remote-as 64512`  
   `set protocols bgp 64512 neighbor 10.0.0.26 remote-as 64512`
4. Finally, commit and save the config:  
   `commit`  
   `save`

You can check to see if BGP as come up with `show ip bgp`:

    admin@USG:~$ show ip bgp
    No BGP network exists

Its OK that no BGP network exist yet because we haven't configure the MetalLB side or exposed any `Services` yet.  Lets do that now.

I'll assume that you've[ followed my other tutorial on MetalLB](https://blog.cowger.us/2018/07/25/using-kubernetes-externaldns-with-a-home-bare-metal-k8s.html) - if not, do that now.

We need to configure a second address pool with BGP for MetalLB, and we'll do that by editing the `ConfigMap`.  The default from the previous tutorial looked a little like this:

    ---
    apiVersion: v1
    kind: ConfigMap
    metadata:
      namespace: kube-system
      name: metallb-config
    data:
      config: |
        address-pools:
        - name: default
          protocol: layer2
          addresses:
          - 10.0.0.200-10.0.0.220
          avoid-buggy-ips: true

And thats fine for dishing out IPs in the 10.0.0.200-10.0.0.220 range.  But to use a dedicated range (like 10.1.0.0/24) we'll need something a bit different.  First, we'll add a section of 'peers' to define who the MetalLB speakers should talk to (hint: its our router):

        peers:
        - peer-address: 10.0.0.1
          peer-asn: 64512
          my-asn: 64512

Again, we use the private AS number, and specify the peer as the router's IP address.

Next, we add an address pool for BGP:

        - name: bgp
          protocol: bgp
          addresses:
          -  10.1.0.0-10.1.0.254
          avoid-buggy-ips: true

And apply that `ConfigMap` with our regular methods (`kubectl apply -f ...`).

I've added the option to `avoid-buggy-ips` just because some routers are goofy about the first and last IP in a range.

Once we do that, we can check our router to see if its discovered these peers yet with `show ip bgp neighbours` - you'll get output for each of the peers you created, and you are looking for something that says the session is established:

    admin@USG:\~$ show ip bgp neighbors
    BGP neighbor is 10.0.0.24, remote AS 64512, local AS 64512, internal link
      BGP version 4, remote router ID 10.0.0.24
      BGP state = Established, up for 00:29:17

Excellent!  Now lets make sure we have a `Service` of type `LoadBalancer`, and then request an IP in the newly defined range, by editing an existing `Service` to add:

    type: LoadBalancer
    
    loadBalancerIP: 10.1.0.1

If you ask k8s for the Service info (`kubectl get svc kubernetes-dashboard`) you'll see we got the IP we requested:

    $ kubectl get svc kubernetes-dashboard
    NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
    kubernetes-dashboard   LoadBalancer   10.100.206.131   10.1.0.1      443:31839/TCP   2d3h

And we can even `curl` it successfully:

    ╰ curl https://10.1.0.1/ -k
     <!doctype html> <html ng-app="kubernetesDashboard"> <head> <meta charset="utf-8"> <title ng-controller="kdTitle

We can also check on the BGP peering session from the router:

    admin@USG:~$ show ip bgp
    BGP table version is 0, local router ID is 10.0.0.1
    Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
                  r RIB-failure, S Stale, R Removed
    Origin codes: i - IGP, e - EGP, ? - incomplete
    
       Network          Next Hop            Metric LocPrf Weight Path
    *>i10.1.0.1/32      10.0.0.24                       0      0 ?
    * i                 10.0.0.26                       0      0 ?
    
    Total number of prefixes 1

We have 1 prefix of size `/32` (e.g. 1 IP address) available via either 10.0.0.24 or 10.0.0.26, with 10.0.0.24 currently preferred (the `>` notation).

I love it!   Feel free to share if you got this working in your lab!