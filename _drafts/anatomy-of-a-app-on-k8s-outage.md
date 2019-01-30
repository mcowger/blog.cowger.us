---
title: Anatomy of a App-on-K8s OUtage
date: 2019-01-30 00:00:00 -0800

---
A lot of the work I do with customers centers around helping them become successful, as quickly as possible.

For some customers that's helping install, for some its helping with pipelines, and for others its just help containerizing applications and getting them ready to run on Kubernetes (and all that entails).

This is the story of the first outage one application took while on K8s, the root cause, and maybe how we'll fix it.   Fair warning - I can't disclose the customer, or the application.

**The Setup**

This application is a scale out Java application, running on-premises for the customer on Pivotal PKS.  It receives requests from outside K8s, makes appropriate database calls (against a DB also outside K8s), and returns results.   Its accessed via an Ingress, and has a standard Service endpoint. _The one goofy thing about this application is that it does a ton of processing on startup to warmup its cache, etc, and so its about 20-24 minutes worth of startup time before it can take requests._  That's fine - we can deal with that via liveness and readiness checks.

It was running fine for weeks, scaled out with a Deployment to 30-ish Pods, etc.

**The Challenge**

The underlying cluster (about 8 nodes) needed to be upgraded from PKS 1.2.6 to 1.3.   Now, because the customer was running PKS, that's a mostly hands off process...

* Initiate upgrade
* BOSH does the rest:
  * Node by node:
    * Drain + Cordon the Node
    * Delete the node from the IaaS
    * Create new Node from new image on IaaS
    * Add node to cluster

Now in PKS, each node really only takes about 3-4 minutes to do this, and its all handled by the system.

**The Event**

About 25 minutes after the upgrade was started, the application monitoring started noticing that transactions were failing - like 95%+ of transactions to the application were timing out.   After about 27 minutes, the application was entirely unresponsive.

What happened?   Kubernetes should have migrated those Pods off the nodes as they were drained and restarted elsewhere.

**_STOP HERE TO THINK AND FIGURE OUT WHY_**

**The Root Cause**

The core the problem is the 25 minute startup time.  Each of the 8 nodes in the cluster ran about 3 of the Pods in the deployment.   When the first Node was drained and its \~3 Pods evicted, they restarted on other Nodes, but needed to perform their 25 minute startup sequence.

After 3-4 minutes of working on the first node, the second node experienced the same fate, with its \~3 Pods being evicted and beginning their startup sequence.  Even worse, probably some of these Pods on the second node were ones from the first node that still hadn't started, and just got killed again.

Roll through the entire cluster in about 20 minutes, and you end up in a situation where all the Pods are _executing_ per the Deployment's design, but none of them are ready/live (and therefore aren't part of the Service or Ingress yet).   This resulted in the complete failure of the application.

**The Fix**

There is a technique in Kubernetes to prevent this.  What we really want to be able to say is "Don't drain this node if the overall application would be overly impacted".   This is measured (generally) as a disruption budget - how many components of this application can be down simultaneously without impacting our SLA.

For this particular application, thats about 70% - we can suffer a loss of about 70% of the Pods without a significant negative impact.

The way that Kubernetes handles this is with a Pod Disruption Budget, or PDB, where we can codify that and apply it to any given set of selectors.

In this case, the fix was to create a PDB that looks something like this:

    apiVersion: policy/v1beta1
    kind: PodDisruptionBudget
    metadata:
      name: app-pdb
    spec:
      maxUnavailable: 30%
      selector:
        matchLabels:
          app: myappname

With this in place, as a drain process starts, it will check the current state of all the Pods aligned to that selector (`app: myappname`) and will wait to drain the node until there are enough Pods to stay below that `maxUnavailable` value of 30%.

**Conclusion**

Fortunately, in this case, by the time the last node was upgraded/replaced, the first node's Pods were nearly complete with their startup, so the outage lasted only about 14 minutes.

It was a fairly inexpensive lesson for the team to do a couple things differently:

1. Do larger scale destructive testing - rather than the basic tests they had tried of killing a couple Pods manually, the team needs to work through entire AZ failures and upgrades in the staging environment as part of their release QA process.
2. Document the Best Practices like PDBs in a central place
3. Figure out how to reduce the startup time of their application.