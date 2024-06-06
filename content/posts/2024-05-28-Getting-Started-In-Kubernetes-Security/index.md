---
title: "Getting Started in Kubernetes Security"
date: 2024-05-28T14:05:06+01:00
draft: true
---

This is a post I've been meaning to write for a while. It's not going to be a definitive guide, more a ramble through some of the resources available to those looking to start down this path. Huge thanks to everyone who attended the DevSecCon London meetup last week to participate in our CTF, for reminding me to write this. 

There's a lot of information out there around Container Security, and none of it is going to be enough to make you an expert in isolation. I've tried to categorise the things I refer to most often below as a starter. I should also say that there's no substitute for just playing with a technology. If you can get an employer to pay for this that's even better. One day I'll write my thoughts on having a homelab, which boil down to "they're great until they cause you more stress than provide help".

I'm also assuming that if you're reading this, you already have an interest in information security or currently work in IT and are looking to move into the container space. If not, sorry, this post probably isn't for you.

## Courses

I'll lead off with my own workshop, regularly delivered with [Rory McCune](https://x.com/raesene/): [Container Security Workshop](/talks/2023-07-07-steelcon-container-security-workshop/). We aim for this to be in introductory 2-4 hour session, although this isn't a lot of time for even a quick jaunt through a topic of this scale. Slides are available on this site, but for full value, try to catch us at a conference near you.

While it's not an intro-level cert in my experience, the CKA is also useful to aim for. [This Udemy course](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests) was useful when I sat mine.

For anything more in-depth, various companies deliver dedicated training. The only two I have direct experience with are my current employer, [ControlPlane](https://control-plane.io/training/) and my former employer, [NCC Group](https://www.nccgroup.com/).

## Books

While anything that's tattooed onto a dead tree is instantly outdated, these are worth reading for both an overview of the fundamentals and some insight into historical security. 

Amazon links below aren't affiliate links or anything. I'm not that cool.

- Hacking Kubernetes - Andrew Martin & Michael Hausenblas https://www.amazon.co.uk/Hacking-Kubernetes-Threat-Driven-Analysis-Defense/dp/1492081736
- Container Security - Liz Rice https://www.amazon.co.uk/Container-Security-Fundamental-Containerized-Applications/dp/1492056707/

## Blogs

- [Raesene's Ramblings](https://raesene.github.io) for things from Rory's brain
- [Container Security Site](https://container-security.site) for a collection of useful further reading and notes. Also maintained by Rory.
- [This Roadmap](https://roadmap.sh/devops) and the associated Kubernetes maps
- Gitlab's resources, specifically [Kubernetes Terminology](https://about.gitlab.com/blog/2020/07/30/kubernetes-terminology/) and [A beginner's guide to container security](https://about.gitlab.com/topics/devsecops/beginners-guide-to-container-security/)
- [Julia Evans' posts](https://jvns.ca/#kubernetes---containers) including some [brilliant comics](https://wizardzines.com/zines/containers/)
- [Ian Lewis](https://www.ianlewis.org/en/) has some great content, including a deep dive into how Container Runtimes work
- [Skybound](https://www.skybound.link/) for CTF writeups and other security musings

## Talks/Videos

There are way too many of these to really list, but I'll give it a go.

- Anything from [https://talks.container-security.site/](https://talks.container-security.site/) for security-related content
- [Liz Rice - Containers From Scratch](https://www.youtube.com/watch?v=8fi7uSYlOdc)
- [Advanced Persistent Threats](https://www.youtube.com/watch?v=CH7S5rE3j8w) is definitely not an introductory talk, but it's an excellent example of what could happen if you aren't secure.
- 

## Tools


