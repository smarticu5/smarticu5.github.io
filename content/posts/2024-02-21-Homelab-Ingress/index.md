---
title: "Homelab Ingress and ExternalNames"
tags:
- homelab
- kubernetes
summary: Making Kubernetes handle homelab networking and DNS.
date: 2024-02-21T00:00:00Z
---

## Homelab Networking

My home network is roughly split into VLANs with a limited amount of network isolation. As usual there's an eternal TODO item to split things out properly, but for now it's basically split into `IoT`, `Servers`, and `LAN`. For the purposes of this post I'll only focus on `LAN` and `Servers`. 

I run `home.domain.com` for my apps hosted in a RKE2 Kubernetes cluster, with wildcard DNS pointing to the ingress point for that cluster. This means I can create a new app, `app1.home.domain.com`, and get a Kubernetes Ingress resource pointing at a service of my choice. This lets me have Kubernetes via Cert-Manager handle my TLS as well. Incidentally, I'm using the Lets Encrypt DNS01 challenge with Route53 to handle certificates, so if I'm ever not at home I still get DNS responses the servers just don't respond.

Last week I deployed [UptimeKuma](https://github.com/louislam/uptime-kuma) as a dashboard, hosted as an add-on to Home Assistant. This meant it was outside my Kubernetes cluster, as Home Assistant runs on its own machine. I also run a couple of other services extra-cluster, and until recently handled their own TLS manually. This meant running cronjobs on a schedule on each device, and issuing a number of certificates which were likely not fully required. 

After some digging I realised that Kubernetes supports [ExternalName Services](https://kubernetes.io/docs/concepts/services-networking/service/#externalname), which are basically in-cluster CNAME records. This means you can create a service in cluster which can be reached through your ingress/load balancer of choice, and have cert-manager still provision certificates. There's still technically cleartext or non-trusted TLS happening between the cluster and that end service but for my homelab non-production needs, this is an accepted risk. 

Setting this up was relatively trivial. I created a service that points to my Home Assistant Device:

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: homeassistant
  name: homeassistant
  namespace: apps
spec:
  externalName: homeassistant.lan.domain.com
  selector:
    app: homeassistant
  type: ExternalName
status:
  loadBalancer: {}
```

Then I made an Ingress Resource which points to that service. As no port name was specified in the service, we need to make sure we get the right port in Ingress:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    cert-manager.io/issuer: letsencrypt-homelab
    cert-manager.io/issuer-kind: ClusterIssuer
    nginx.ingress.kubernetes.io/rewrite-target: /
  name: uptime
  namespace: apps
spec:
  rules:
  - host: uptime.home.domain.com
    http:
      paths:
      - backend:
          service:
            name: homeassistant
            port:
              number: 3001
        path: /
        pathType: Prefix
  tls:
  - hosts:
    - uptime.home.domain.com
    secretName: wildcard-domain-com-tls
status:
  loadBalancer:
    ingress:
    - ip: x.y.z.1
    - ip: x.y.z.2
    - ip: x.y.z.3
```

It's worth noting that I use a wildcard cert, just so I don't need to get a new cert from LetsEncrypt every time I create a new service. 

This approach lets me easily add easy-to-remember hostnames for apps in the house, rather than needing to remember that application X is hosted on device Y. It means if I create a new app running under Unraid or something that needs its own hardware, I can still keep the ingress consistent, and makes it way easier to add bookmarks and maintain dashboards etc. 