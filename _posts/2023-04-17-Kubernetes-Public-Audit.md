---
layout: post
title:  "Kubernetes 1.24 Public Audit"
---

Originally posted at: https://research.nccgroup.com/2023/04/17/public-report-kubernetes-1-24-security-audit/

NCC Group was selected to perform a security evaluation of Kubernetes 1.24.0 release in response to Kubernetes SIG Security’s Third-Party Security Audit [Request for Proposals](https://github.com/kubernetes/sig-security/blob/main/sig-security-external-audit/security-audit-2021-2022/RFP.md). The testing portion of the audit took place in May and June 2022. The global project team performed a security architectural design review that resulted in the identification of findings in terms of secure design of Kubernetes. The team also performed dynamic native application pen tests, including source code and cryptographic review which found vulnerabilities in multiple components. 

Key findings included: 

- Concerns with the administrative experience 
- Flaws in communication between the API Server and the Kubelet which may result in an elevation of privilege 
- Flaws in input sanitization which provide a limited authorization bypass (publicly disclosed under [CVE-2022-3162](https://github.com/kubernetes/kubernetes/issues/113756)) 

