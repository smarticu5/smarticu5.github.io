---
layout: post
title: Kubernetes imagePullSecrets
categories: [Kubernetes, Docker]
---

# Image Pull Secrets

Woo, first post! I've been looking more into container security lately and my friends and family are bored of hearing me talk about it, so I'm turning to a random internet audience now. More likely no one will read this, and I'll end up using it as self-reference.

Kubernetes clusters are used to run images. Sure, they do some fancy witchcraft to make them run in a distributed, high availability manner, but none of that matters if your cluster can't get your images from a registry somewhere. Normally, this is pretty simple. In a test cluster using [kind](https://github.com/kubernetes-sigs/kind), let's create a really basic manifest to deploy a pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels:
    name: myapp
spec:
  containers:
  - name: myapp
    image: smarticu5/demo_public
    ports:
      - containerPort: 80
```

Running `kubectl apply -f manifest.yaml`, the pod is created and we have a webserver running. 

```bash
❯ kubectl get po
NAME    READY   STATUS    RESTARTS   AGE
myapp   1/1     Running   0          107s
❯ kubectl get events
108s        Normal    Pulling   pod/myapp   Pulling image "smarticu5/demo_public"
57s         Normal    Pulled    pod/myapp   Successfully pulled image "smarticu5/demo_public"
57s         Normal    Created   pod/myapp   Created container myapp
57s         Normal    Started   pod/myapp   Started container myapp
```

The image mentioned above, `smarticu5/demo_public`, is on the Docker Hub as a public image. You can pull it if you want to; all web development job offers off the back of this are welcome. The nodes (node, in this case) of my cluster had no idea about the image before the manifest was deployed, so they had to pull the image from the hub. The image was pulled, the processes started, everyone's happy. 

## Private images

But what would we do if the image was private? Let's say I had super secret reasons to make a private image, but still wanted to run it in my cluster. I've got a test image in a private repo on the Docker Hub, as `smarticu5/test_private_repository`. If a manifest is deployed using the second image, we get some really helpful errors. 

```yaml
...snip...
  containers:
  - name: secretapp
    image: smarticu5/test_private_repo
...snip...
```

```bash
❯ kubectl get po
NAME        READY   STATUS             RESTARTS   AGE
myapp       1/1     Running            1          63m
secretapp   0/1     ImagePullBackOff   0          26s

```

The error `ImagePullBackOff` helpfully tells us that the container runtime (Docker) isn't able to pull the image. Normally this is because of a typo, but in this case it's because the repository is private. Kubernetes needs a way to authenticate to the Docker Hub to pull the image. 

## Enter imagePullSecrets

Obviously, this problem has a few solutions. The simplest one is the use of imagePullSecrets, which are a Kubernetes secret of type `docker-registry` and solve this exact issue. If we were trying to pull the image to a developer laptop we'd perform a `docker login` then use the generated token to authenticate, so let's do the same in Kubernetes. 

First, we need to create the secret itself:

```bash
kubectl create secret docker-registry dockerhub --docker-server=https://index.docker.io/v1/ --docker-username=<smarticu5> --docker-password=<ObviouslyThisIsntMyRealPassword> --docker-email=<howdoemailswork@test.com>
```

We can confirm the secret by running the following:

```bash
❯ kubectl get secret dockerhub -o yaml
apiVersion: v1
data:
  .dockerconfigjson: eyJhdXRocyI6eyJodHRwczovL2luZGV4LmRvY2tlci5pby92MS8iOnsidXNlcm5hbWUiOiJzbWFydGljdTUiLCJwYXNzd29yZCI6IjxEaWRZb3VUaGlua0lGb3Jnb3RUb0NoYW5nZUl0SGVyZT8+IiwiZW1haWwiOiJpLnNtYXJ0OTRAZ21haWwuY29tIiwiYXV0aCI6ImMyMWhjblJwWTNVMU9raHZkMEZpYjNWMFNHVnlaVDg9In19fT09
kind: Secret
metadata:
  creationTimestamp: "2020-04-23T19:59:04Z"
  name: dockerhub
  namespace: default
  resourceVersion: "17895"
  selfLink: /api/v1/namespaces/default/secrets/dockerhub
  uid: a4e4852a-02da-4f0f-93a7-8331ad40d842
type: kubernetes.io/dockerconfigjson
```

Then we can modify our manifest to make use of the secret, and authenticate to pull down the image:

```yaml
spec:
  containers:
  - name: secretapp
    image: smarticu5/test_private_repository
    ports:
      - containerPort: 80
  imagePullSecrets:
  - name: dockerhub
```

Now when we check on the cluster status, the image has been pulled and the images are running successfully. 

```bash
❯ kubectl get po
NAME        READY   STATUS    RESTARTS   AGE
myapp       1/1     Running   0          22s
secretapp   1/1     Running   0          22s
```

We've now been able to pull a private image from a repository, but everything so far has been running as a highly priviliged cluster-admin role. Coming soon: how to make this play nicely with RBAC. 