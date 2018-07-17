---
title: A Read Only Kubernetes Dashboard
date: 2018-07-03 07:00:00 +0000

---
I've been working extensively with a new PKS customer recently, and one of the challenges we face is helping newbies understand whats happening in their K8S cluster.  One of the best ways to do that is with 'anchoring' - giving them something that _looks_ like what they know, so they can start to relate.

One of the ways we've done that is with the Kubernetes Dashboard - having something that _looks_ familiar has been valuable to get engineers to a state where they feel comfortable about whats happening as part of their `kubectl apply` sequences.  It certainly looks nice:

![](https://d33wubrfki0l68.cloudfront.net/e6bda94ebf94cc460db5cdc42bbfdb8f95f5f7ce/fd28b/images/docs/ui-dashboard.png)

However, there's a challenge...the default configuration for the dashboard does not give it permissions to see anything within the cluster.  The dashboard page acknowledges this and suggests simply binding the dashboard service account to cluster-admin...YIKES:

    apiVersion: rbac.authorization.k8s.io/v1beta1
    kind: ClusterRoleBinding
    metadata:
      name: admin-user
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: cluster-admin
    subjects:
    - kind: ServiceAccount
      name: admin-user
      namespace: kube-system

Ugh!  Thats not OK.   Better would be a read only user...like the 'view' ClusterRole:

![](/uploads/Screen Shot 2018-07-17 at 4.49.47 PM.png)But there's a problem - the _view_ ClusterRole doesn't actually have permissions for the Cluster level objects like Nodes and Persistent Volume Claims.  So we'll have to create a new RBAC config.

First, we'll create a new **dashboard-viewonly** ClusterRole:

```yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: dashboard-viewonly
rules:
- apiGroups:
  - ""
  resources:
  - configmaps
  - endpoints
  - persistentvolumeclaims
  - pods
  - replicationcontrollers
  - replicationcontrollers/scale
  - serviceaccounts
  - services
  - nodes
  - persistentvolumeclaims
  - persistentvolumes
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - bindings
  - events
  - limitranges
  - namespaces/status
  - pods/log
  - pods/status
  - replicationcontrollers/status
  - resourcequotas
  - resourcequotas/status
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - namespaces
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - apps
  resources:
  - daemonsets
  - deployments
  - deployments/scale
  - replicasets
  - replicasets/scale
  - statefulsets
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - autoscaling
  resources:
  - horizontalpodautoscalers
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - batch
  resources:
  - cronjobs
  - jobs
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - extensions
  resources:
  - daemonsets
  - deployments
  - deployments/scale
  - ingresses
  - networkpolicies
  - replicasets
  - replicasets/scale
  - replicationcontrollers/scale
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - policy
  resources:
  - poddisruptionbudgets
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - networking.k8s.io
  resources:
  - networkpolicies
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - storage.k8s.io
  resources:
  - storageclasses
  - volumeattachments
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - rbac.authorization.k8s.io
  resources:
  - clusterrolebindings
  - clusterroles
  - roles
  - rolebindings
  verbs:
  - get
  - list
  - watch
```

And then bind it to the service account:

```yaml
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard
  labels:
    k8s-app: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: dashboard-viewonly
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard
  namespace: kube-system
```

Now we've configured a readonly dashboard we can safely share with people that dont even have K8S cluster access (many devs dont - they just push via a CI/CD pipeline).   As a side note, this RBAC config does _NOT_ grant access to Secrets, just to be a little safe.

And voila:

![](/uploads/Screen Shot 2018-07-17 at 3.05.35 PM.png)