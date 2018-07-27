---
title: 'Using Kubernetes ExternalDNS & MetalLB with a Home/Bare Metal K8S: Part 2'
date: 2018-07-26 00:00:00 -0700

---
[In my previous post](https://blog.cowger.us/2018/07/25/using-kubernetes-externaldns-with-a-home-bare-metal-k8s.html), I described the method I used with MetalLB to expose Kubernetes Services with LoadBalancer stype virtual IPs within my home lab.

In today's post, I'll take that one step further and attach dynamically managed DNS entries to those VIPs as well.

Fortunately, there's an incubated projects from the Kubernetes team called 'external-dns' that does this for me and it supports a wide variety of DNS providers, including all the major cloud providers, as well as on prem providers like PowerDNS and Infoblox.  I personally use Google Cloud DNS to manage mine, and its on the list.

In theory, this should be super simple....just deploy the Deployment

```yaml

    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      name: external-dns
    spec:
      strategy:
        type: Recreate
      template:
        metadata:
          labels:
            app: external-dns
        spec:
          serviceAccountName: external-dns
          containers:
          - name: external-dns
            image: registry.opensource.zalan.do/teapot/external-dns:latest
            args:
            - --source=service
            - --source=ingress
            - --domain-filter=external-dns-test.gcp.zalan.do # will make ExternalDNS see only the hosted zones matching provided domain, omit to process all available hosted zones
            - --provider=google
    #        - --google-project=zalando-external-dns-test # Use this to specify a project different from the one external-dns is running inside
            - --policy=upsert-only # would prevent ExternalDNS from deleting any records, omit to enable full synchronization
            - --registry=txt
            - --txt-owner-id=my-identifier
            
```

But its not quite so simple.  The design above assumes that you are running _ON GCP_ and therefore have builtin access to all the GCP credentials.   In my case I'm not hosting my cluster GCP, but in my home lab and just using Google for DNS.   I need some way to include my GCP credentials in this deployment.

Step was was to uncomment the `google project` option on line 23:

`- --google-project=home-stuff-44444`

But I still needed a way to get my credentials in there.  I decided the best way was to make use of the fact that the credentials can come from environment variables:

```yaml
        env:
        - name: GOOGLE_APPLICATION_CREDENTIALS
          value:  /var/run/secrets/kubernetes.io/gcloud-config/google.js
```

But where did that file come from? `/var/run/secrets/kubernetes.io/gcloud-config/google.js`?

Well Kubernetes has a way to store 'secret' information, and make it available to containers in all sorts of formats.  One of them is to create a secret with data from a file, and then simply expose that 'secret' as though it were a real file in the container's filesystem.  Its a neat trick.

I created the secret simply using my Google service account's JSON key file with `kubectl create secret generic gcloud-config --from-file=google.js`

From there, I mounted it into the container with a volume and volumeMount defintions:

            volumeMounts:
            - mountPath: /var/run/secrets/kubernetes.io/gcloud-config
              name: gcloud-config
              readOnly: true

Which was really pretty easy, then defining a volume:

          volumes:
          - name: gcloud-config
            secret:
              secretName: gcloud-config

Now my container can read this, and if I ever update it, the container gets updated right along with it.

From here, its a matter of adding annotations to services to get external-dns to set them up:

    apiVersion: v1
    kind: Service
    metadata:
      name: nginx
      annotations:
        external-dns.alpha.kubernetes.io/hostname: nginx.home.cowger.us.

Super cool!