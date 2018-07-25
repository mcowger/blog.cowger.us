---
title: A Basic Validating Admission Controller Webhook for Kubernetes
date: 2018-07-24 00:00:00 -0700

---
I've been working recently with a customer that has a couple interesting needs within their Kubernetes clusters:

1. They need privileged containers enabled on the cluster as a whole to support various kinds of[ monitoring tools](https://www.outcoldsolutions.com/docs/monitoring-kubernetes/v4/) and storage persistence tools (like [ScaleIO](https://github.com/VxFlex-OS/charts/tree/master/vxflex-csi), [PortWorx](https://github.com/portworx/helm) and [Pure Storage](https://github.com/purestorage/helm-charts)).  However, they also don't want developers to deploy privileged containers for no reason.  Currently, [PodSecurityPolicies](https://kubernetes.io/docs/concepts/policy/pod-security-policy/) are not an option because they aren't available in PKS at the moment.
2. They need the ability to inject various options into a container as its deployed in case the developers set an incorrect value (or a just missing the value).   Think of things like `PROXY_HOST` options.

So after some brainstorming with the always awesome [Merlin](https://twitter.com/virtualmerlin?lang=en) from VMware, we came up with the idea of using Mutating Webhooks to solve both of these problems.

After one registers their webhook with Kubernetes as an AdmissionController, all requests for Pods to be deployed (whether directly or via Deployments, ReplicaSets, etc).  The Webhook can then accept the request, deny the request or, possibly, modify the request.

This customer had asked for a sample of what that might look like, so I wrote up a quick one that validates that a Pod has no environment variables set.  

The request that comes in looks something like this:

```json
{'apiVersion': 'admission.k8s.io/v1beta1',
 'kind': 'AdmissionReview',
 'request': {'kind': {'group': '', 'kind': 'Pod', 'version': 'v1'},
             'namespace': 'default',
             'object': {'metadata': {'creationTimestamp': '2018-07-17T18:09:56Z',
                                     'labels': {'run': 'busybox'},
                                     'name': 'busybox',
                                     'namespace': 'default',
                                     'uid': '9c6bf542-89ec-11e8-9284-52110844bbb0'},
                        'spec': {'containers': [{'image': 'busybox',
                                                 'imagePullPolicy': 'IfNotPresent',
                                                 'name': 'busybox',
                                                 'resources': {},
                                                 'terminationMessagePath': '/dev/termination-log',
                                                 'terminationMessagePolicy': 'File',
                                                 'volumeMounts': [{'mountPath': '/var/run/secrets/kubernetes.io/serviceaccount',
                                                                   'name': 'default-token-h978h',
                                                                   'readOnly': "TRUE"}]}],
                                 'dnsPolicy': 'ClusterFirst',
                                 'restartPolicy': 'Never',
                                 'schedulerName': 'default-scheduler',
                                 'securityContext': {},
                                 'serviceAccount': 'default',
                                 'serviceAccountName': 'default',
                                 'terminationGracePeriodSeconds': 30,
                                 'tolerations': [{'effect': 'NoExecute',
                                                  'key': 'node.kubernetes.io/not-ready',
                                                  'operator': 'Exists',
                                                  'tolerationSeconds': 300},
                                                 {'effect': 'NoExecute',
                                                  'key': 'node.kubernetes.io/unreachable',
                                                  'operator': 'Exists',
                                                  'tolerationSeconds': 300}],
                                 'volumes': [{'name': 'default-token-h978h',
                                              'secret': {'secretName': 'default-token-h978h'}}]},
                        'status': {'phase': 'Pending',
                                   'qosClass': 'BestEffort'}},
             'oldObject': "NULL",
             'operation': 'CREATE',
             'resource': {'group': '', 'resource': 'pods', 'version': 'v1'},
             'uid': '9c6bf8fc-89ec-11e8-9284-52110844bbb0',
             'userInfo': {'groups': ['system:masters', 'system:authenticated'],
                          'username': 'minikube-user'}}}
```

We can look at any of these fields to make a decision to admit the Pod or not.   In my case, with Python, the code was very simple:

```python
### Library Includes and Boilerplate ###
from flask import Flask, request, jsonify
from pprint import pprint
import json
app = Flask(__name__)


@app.route('/', methods=['POST']) ###Listen on / for POST requests
def webhook():
    allowed = True #Default to allowed
    request_info = request.json #read the JSON into a Python dict
    for container_spec in request_info["request"]["object"]["spec"]["containers"]: #For each container defined in the request
        if 'env' in container_spec: #if there are environment variables set....
            print("Environment Variables Cannot Be Passed to Containers")
            allowed = False #NOPE!

	# Now construct the response JSON
    admission_response = {
        "allowed": allowed
    }
    admissionReview = {
        "response": admission_response
    }
    return jsonify(admissionReview) #And send it back!


app.run(host='0.0.0.0', debug=True)

```
And what we send back to the request is pretty short and sweet:

```json
{"response": {"allowed": false}}
```
Installing the webhook is as simple as finding a place to host it (you need to enable SSL - Kubernetes doesn't want to do it without SSL), and then adding the yaml to register it:

```yaml
apiVersion: admissionregistration.k8s.io/v1beta1
kind: ValidatingWebhookConfiguration
metadata:
  name: denyenv
webhooks:
  - name: denyenv.pivotal.io
    rules:
      - apiGroups:
          - "*"
        apiVersions:
          - v1
        operations:
          - CREATE
        resources:
          - pods
    failurePolicy: Fail
    clientConfig:
      url: "https://webhook.url/"
```

