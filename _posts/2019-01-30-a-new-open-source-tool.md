---
title: A New Open Source Tool
date: 2019-01-30 00:00:00 -0800

---
Almost a year ago I started working for [Pivotal](https://pivotal.io/) on their Kubernetes distribution, [PKS (Pivotal Container Services).](https://pivotal.io/platform/pivotal-container-service)

Over that time I've helped a number of customers, and one thing that came up a few times was how to easily generate a `kubeconfig` for L[DAP/UAA backe OIDC authentication](https://kubernetes.io/docs/reference/access-authn-authz/authentication/).  The built in `pks` CLI doesn't do it, and[ Dex/Gangway doesn't currently support the lagre cookies or blank client secrets we use in UAA.  ](https://github.com/heptiolabs/gangway/pull/85)

A [colleague had built a quick shell script](https://community.pivotal.io/s/article/script-to-automate-generation-of-the-kubeconfig-for-the-kubernetes-user) to do this, but it was a bit fragile and didn't work well on Windows.   My customers asked for something like a binary that would work across platforms, have no dependencies and be easy to distribute.

So, over the Holiday break, I learned just enough Go to make that work, and wrote `pkstoken`.

After sharing it with some colleagues, I worked with Pivotal's legal team to get this open sourced.  So without further ado: [PKSToken is live.](https://github.com/pivotal/pkstoken)

There's full descriptions of how to use it on the GitHub [README](https://github.com/pivotal/pkstoken/blob/master/README.md), but its pretty simple:

`kubectl-pkstoken -api=api.pks.mydomain.com -cluster=ldap.pks.exaforge.com -user=euler -ns=default -kubeconfig=myconfig`

Enjoy!