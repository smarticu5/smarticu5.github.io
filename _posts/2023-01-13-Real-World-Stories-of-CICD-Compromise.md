---
title: "10 real-world stories of how we’ve compromised CI/CD pipelines"
---

Originally posted by NCC Group at [https://research.nccgroup.com/2022/01/13/10-real-world-stories-of-how-weve-compromised-ci-cd-pipelines/]() with multiple authors.


Mainstream appreciation for cyberattacks targeting continuous integration and continuous delivery/continuous deployment (CI/CD) pipelines has been gaining momentum. Attackers and defenders increasingly understand that build pipelines are highly-privileged targets with a substantial attack surface.

But what are the potential weak points in a CI/CD pipeline? What does this type of attack look like in practice? NCC Group has found many attack paths through different security assessments that could have led to a compromised CI/CD pipeline in enterprises large and small.

In this post, we will share some of our war stories about what we have observed and been able to demonstrate on CI/CD pipeline security assessments, clearly showing why there is the saying, “_they are execution engines_.”

Through showing many different flavors of attack on possible development pipelines, we hope to emphasize the criticality of securing this varied attack surface to better secure the software supply chain.

## Jenkins with multiple attack angles

The first 3 attack stories are related to [Jenkins](https://www.jenkins.io/), a leading tool in CI/CD used by many companies and one of our consultants came across when working on multiple assessments for major software companies.   

### Attack #1: “It Always Starts with an S3 Bucket…”

The usual small misconfiguration in an S3 bucket led to a full DevOps environment compromise. The initial attack angle was via a web application. The attack flow for this compromise involved:

`**Web application -> Directory listing on S3 bucket -> Hardcoded Git credential in script file -> Git access -> Access Jenkins with the same hardcoded Git credential -> Dump credentials from Jenkins -> Lateral movement -> Game Over -> Incident -> Internal Investigation**`

NCC Group performed a black box web application assessment with anonymous access on an Internet-facing web application. At the beginning of the test, a sitemap file was discovered in a sitemap folder. The sitemap folder turned out to be an S3 bucket with directory listing enabled. Looking through the files in the S3 bucket, a bash shell script was spotted. After a closer inspection, a hardcoded git command with a credential was revealed. The credentials gave the NCC Group consultant access as a limited user to the Jenkins Master web login UI which was only accessible internally and not from the Internet. After a couple of clicks and looking around in the cluster they were able to switch to an administrator account. With administrator privileges, the consultant used Groovy one-liners in the script console and dumped around 200 different credentials such as AWS Access token, SAST/DAST tokens, EC2 SSH certificates, Jenkins users, and other Jenkins credentials. The assessment ended with the client conducting incident response working closely with the consultant for remediation.

NCC gave a detailed report with remediation and hardening steps for the client, and some of the recommended steps were the following:

- Remove directory listing for S3
- Remove shell script file and hardcoded credential
- Remove the connection that allows whoever has GitHub access can access Jenkins
- Install and review Audit Trail and Job Configuration History plugins
- Jenkins should not be accessible from the Internet in this case we tested onsite
- Change and lower the privileges the Jenkins account had
- Deploy and use MFA for administrator accounts

### Attack #2: Privilege Escalation in Hardened Environment

The following steps describe another privilege escalation path found by the consultant on a different assessment:

**`Login with SSO credentials -> Testing separated, lock down and documented roles -> One role with Build/Replay code execution -> Credentials dump`**

The assessment was to review a newly implemented hardened Jenkins environment with the documented user roles that had been created using the least privileges principle. Jenkins was running under a non root-user, latest version of core and plugins, had SSL certification and SSO with MFA for login. NCC Group consultant had access to one user with a specific role per day and tested if there was any privilege escalation path.

The builder role had the Build/Replay permission as well (that allows replaying a Pipeline build with a modified script Or with additional Groovy code), not just the build permission to build the jobs. This allowed NCC Group consultants to run Groovy code and dump credentials of Jenkins users and other secrets.

### Attack #3: Confusing Wording in a Plugin

The last angle was a confusing option in a Jenkins plugin that led to wide-open access:

`**GitHub Authorization -> Authenticated Git User with Read Access -> Jenkins Access with Gmail Account**`

The GitHub OAuth Plugin was deployed in Jenkins that provided authentication and authorization. The “Grant READ permissions to all Authenticated Users” and “Use GitHub repository permissions” options were ticked that allowed anyone with a GitHub account (even external users) accessing the Jenkins web login UI. NCC was able to register and use their own hosted email account to get access to their projects.

## GitLab CI/CD Pipeline Attacks

NCC Group has done many jobs that have looked at another well-known and used tool called GitLab. As a result, the NCC Group consultant has found some interesting attack paths.

### Attack #4: Take Advantage of Protected Branches

On one particular job, there were multiple major flaws in how the GitLab Runners were set up. The first major flaw was that the runners were using privileged containers, which means they were configured to use the “—privileged” flag that would allow them to spin up other privileged containers that could trivially escape to the host. This one was a pretty straightforward attack vector and could get you to the host. But what made this one interesting was these GitLab Runners were also shared Runners and not isolated. One developer who was only supposed to push code to a certain repository could also get access to secrets and highly privileged repositories. In addition, these shared runners were using trivial environment variables that stored highly sensitive secrets, such as auth tokens and passwords. A user who had limited push access to a repository could be able to get highly privileged secrets.

Protected branches are branches that can be maintained by someone with a maintainer role within GitLab and they can say “only these people can push against these source code repositories or branches” and there is a change request (CR) chain associated with it. These protected branches can be associated with a protected runner. You can lock it down, so the developer has to get the CR approved to push code. But in this case, there was no CR and protected branch implemented and enforced. Anybody could push to an unprotected branch and then chain the previous exploits. Chaining of these 4-5 vulnerabilities gave all access.

There were lots of different paths as well. Even if the “—privileged” flag was used, there was also another path to get to the privileged containers. There was a requirement for the developers to be able to run the docker command. The host’s docker daemon was shared with the GitLab Shared Runner. That led to access to the host and jumping between containers.

The Consultant asked the client to understand and to help remediate the issues, but was keen to understand root causes for what led to these choices being made. Why did they make these configuration choices, and what trade-offs did they consider? What were the more-secure alternatives to these choices; were they aware of these options, and if so, why weren’t they chosen?

The reason that the company wanted privileged containers was to do static analysis on the code that was being pushed. Consultants explained that they should have isolated Runners and not use the shared Runners, and should have further limited access control. This emphasized an important point: It is possible to run privileged containers and to still somewhat limit the amount of sensitive information exposed.

Many of GitLab’s CI/CD security mechanisms to execute jobs depend on the premise that protected branches only contain trusted build jobs and content as administrated by Project’s Maintainers. Users at a Project’s Maintainer Privilege Level or above have the ability to deputize other users to be able to manage and push to specific protected branches as well. These privileged users are the gateway representing what is and is not considered trusted within the project. Making this distinction is important to reduce the exposure of privileges to untrusted build jobs.

### Attack #5: GitLab Runners Using Privileged Containers

On another job, GitLab Runners were configured to execute CI/CD jobs with Docker’s “—privileged” flag. This flag negates any security isolation provided by Docker to protect the host from potentially unsafe containers. By disabling these security features, the container process was free to escalate their privileges to root on the host through a variety of features. Some tools were packaged as Docker images, and to support this, the client used Docker in Docker (DIND) within a privileged container to execute nested Containers.  
  
If privileged CI/CD jobs are necessary, then the corresponding Runners should be configured to only execute on protected branches of projects which are known to be legitimate. This will prevent arbitrary developers from submitting unreviewed scripts which can result in host compromise. The Maintainer Role’s ability to manage protected branches for projects means they regulate control over any privileges supplied by an associated protected Runner.

### Attack #6: Highly Privileged Shared Runners could claim jobs associated with sensitive Environment Variables and Privileged Kubernetes Environments.

Privileged Runners should not be configured as Shared Runners or broadly scoped groups. Instead, they should be configured as needed for specific projects or groups which are considered to have equivalent privilege levels among users at the maintainer level and above.

### Attack #7: Runners Exposed Secrets to Untrusted CI/CD Jobs

Runners make calls to API endpoints which are authenticated using various tokens and passwords. Because these were Shared Runners, authentication tokens and passwords were accessed trivially by any user with rights to commit source code to Gitlab. Runners were configured to expose the secrets through environment variables. Secrets management, especially in CI/CD pipelines is a tough issue to solve.

To mitigate these types of risks, ensure that environment variables configured by Runners in all build jobs do not hold any privileged credentials. Properly-scoped GitLab variables can be used as a replacement. Environment variables should only hold informational configuration values which should be considered accessible to any developer in their associated projects and groups.

If Runners must provide credentials to their jobs through environment variables or mounted volumes, then the Runners should limit the workloads to which they are exposed. To do so, such Runners should only be associated with the most specific possible project/group. Additionally, they ought to be marked as “protected” so that they can only process jobs on protected branches.

### Attack #8: Host Docker Daemon Exposed to Shared GitLab Runner

On one job, the Gitlab Shared Runners mount the Host’s Docker socket to CI/CD job containers at runtime. While this allows legitimate developers to run arbitrary Docker commands on the host for build purposes, it also allows build jobs to deploy privileged containers on the host itself to escape their containment. This also provides attackers with a window through which they can compromise other build jobs running on the host. Essentially, this negates all separation provided by Docker preventing the contained process from accessing other containers and the host. The following remediations are recommended in this case:

- Do not allow developers to directly interact with Docker daemons on hosts they do not control. Consider running Docker build jobs using a tool that supports rootless Docker building such as kaniko.
- Alternatively, develop a process that runs a static set of Docker commands on source code repositories to build them. This process should not be performed within the CI/CD job itself as job scripts are user-defined and the commands can be overwritten.
- If this must be implemented through a CI/CD job, then build jobs executed on these Runners should be considered privileged and as such should be restricted Docker Runners accepting commits made to protected and known safe repositories to ensure that any user-defined CI/CD jobs have gone through a formal approval process.

## Kubernetes

Pods that are running a certain functionality sometimes end up using different pod authentication mechanisms that reach out to various services, AWS credentials are one example. Many times people use plugins and don’t restrict API paths around the plugins. For example, Kube2IAM is a plugin that is seen often and if you don’t correctly configure it from a pod you can get privileged containers that can lead to privileged API credentials that can let you see what the underlying host is doing.

### Attack #9: Kube2IAM

Kube2IAM works off of pod annotations. It intercepts calls from a container pod being made to the AWS API (169254). An NCC Group consultant found an interesting situation where every developer could annotate pods. There was a setting configured with the “sts assume-role *” line in the AWS role that Kube2IAM was using. That allowed any developer who could create/annotate a pod inherits the AWS role of admin. This meant that anyone who could create any pod and specify an annotation could get admin privileges on the main AWS tooling account for a bank. This account had VPC peering configured that could look into any pod and non-pod environments. You could get anywhere with that access. Here is a pipeline that builds a pod, and all an attacker would have to do is add in an annotation to that which outputs something at the end.

There was another similar job performed by an NCC Group consultant. In this scenario, they could not annotate pods – instead, in Kube2IAM there is the flag “whitelist route regex” and you can mention AWS API paths. So you can specify what routes you want to go to/not go to. The DevOps admins had configured that with a white character that would allow someone to get access to privileged paths that would lead to underlying node credentials.

### Attack #10: The Power of a Developer Laptop

In our final scenario, the NCC Group consultant got booked on a scenario-based assessment:

**_“Pretend you have compromised a developer’s laptop.”_**

All that the consultant could do was commit code to a single Java library that was using the Maven project. They set one of the pre-requirement files to an arbitrary file that would give a shell from the build environment. They changed it to a reverse Meterpreter shell payload. They found that the pod had an SSH key lying on disk that went to the Jenkins master node and then dumped all the variables from Jenkins. They then discovered that this was a real deployment pipeline that had write permissions and cluster-admin into the Kubernetes workload. Consequently, they now had access to the full production environment.

There was another job where NCC Group consultant compromised one user account and had access to a pipeline that was authenticated to the developer group. Running custom code was not possible in the pipeline, but they could tell the pipeline to build off a different branch even if it did not exist. The pipeline crashed and dumped out environment variables. One of the environment variables was a Windows domain administrator account. The blue team saw the pipeline crash but did not investigate.

In our final story, NCC Group consultants were on an assessment in which they landed in the middle of the pipeline. They were able to port scan the infrastructure that turned out to be a build pipeline. They found a number of applications with (as of then) unknown functionality. One of the applications was vulnerable to server-side request forgery (SSRF)  and they were running on AWS EC2 instances. The AWS nodes had the ability to edit config maps that allow mapping between an AWS user account and a role inside the cluster. It turned out that this didn’t check if the cluster and the account are in the same user account. As a result, the consultants were able to specify another AWS account to control the clusters and had admin privileges on the Elastic Kubernetes Service cluster (EKS).

## Summary

CI/CD pipelines are complex environments. This complexity requires methodical comprehensive reviews to secure the entire stack. Often a company may lack the time, specialist security knowledge, and people needed to secure their CI/CD pipeline(s).

Fundamentally, a CI/CD pipeline is remote code execution, and must be configured properly.

As seen from above, most compromise has the following root causes or can be traced back to:

- Default configurations
- Over permissive permissions and roles
- Lack of security controls
- Lack of segmentation and segregation