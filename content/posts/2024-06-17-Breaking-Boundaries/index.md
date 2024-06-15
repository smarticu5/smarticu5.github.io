---
title: "Breaking Boundaries"
date: 2024-06-17T00:00:00Z
---

Anyone who's spoken to me for any period of time about Kubernetes, or had the misfortune of being vaguely near me when I'm ranting about it, probably knows my feelings on the setup. It does work, and work effectively, but there are a plethora of sharp edges and unexpected behaviours. A number of these are documented [here](https://kubernetes.io/docs/concepts/security/rbac-good-practices/). This post details my most recent addition to this list. 

RBAC does not claim to solve all security problems in Kubernetes, and indeed it is only one arrow in the proverbial quiver. Multiple controls can be used to supplement RBAC, including a number of security-focussed admission controllers such as OPA Gatekeeper or Kyverno, or the native [Pod Security Admission](https://kubernetes.io/docs/concepts/security/pod-security-admission/). 

Namespaces are intended to provide logical separation between resources. Where any form of multi-tenancy is concerned, tenants' access is often controlled through RBAC and additional policies applied to namespaces directly or through namespace labels. 

Whether they are "officially" a security boundary is a matter of some debate, but the Kubernetes docs state the following about Roles and Rolebindings:

> Roles and RoleBindings are Kubernetes objects that are used at a namespace level to enforce access control in your application

\- [https://kubernetes.io/docs/concepts/security/multi-tenancy/#access-controls](https://kubernetes.io/docs/concepts/security/multi-tenancy/#access-controls)

As a result, many cluster administrators view namespaces as segmented off from each other. On a number of engagements over the last few years, I've reviewed clusters where "tenant administrators", i.e. users who are trusted to have full control over the namespace, have been granted with full RBAC access within that namespace. This is occasionally granted directly through a RoleBinding to the default `cluster-admin` ClusterRole, or through the ability to create your own RBAC entities in the cluster through some form of ID[^1].

[^1]: As CI often runs with cluster-admin permissions, there is rarely a requirement to deal with the [escalate](https://raesene.github.io/blog/2020/12/12/Escalating_Away/) verb. 
