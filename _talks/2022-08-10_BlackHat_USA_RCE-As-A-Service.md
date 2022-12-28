---
title: "RCE-As-A-Service"
excerpt: "Presented at Blackhat USA 2022."
# header:
#   image: /assets/images/foo-bar-identity.jpg
#   teaser: /assets/images/foo-bar-identity-th.jpg
# sidebar:
#   - title: "Role"
#     image: http://placehold.it/350x250
#     image_alt: "logo"
#     text: "Designer, Front-End Developer"
#   - title: "Responsibilities"
#     text: "Reuters try PR stupid commenters should isn't a business model"
# gallery:
#   - url: /assets/images/unsplash-gallery-image-1.jpg
#     image_path: assets/images/unsplash-gallery-image-1-th.jpg
#     alt: "placeholder image 1"
#   - url: /assets/images/unsplash-gallery-image-2.jpg
#     image_path: assets/images/unsplash-gallery-image-2-th.jpg
#     alt: "placeholder image 2"
#   - url: /assets/images/unsplash-gallery-image-3.jpg
#     image_path: assets/images/unsplash-gallery-image-3-th.jpg
#     alt: "placeholder image 3"
---

# RCE-As-A-Service: Lessons Learned from 5 Years of Real World CI/CD Pipeline Compromise

Originally presented at [Blackhat USA 2022](https://www.blackhat.com/us-22/briefings/schedule/#rce-as-a-service-lessons-learned-from--years-of-real-world-cicd-pipeline-compromise-27541)

In the past 5 years, we've demonstrated countless supply chain attacks in production CI/CD pipelines for virtually every company we've tested, with several dozen successful compromises of targets ranging from small businesses to Fortune 500 companies across almost every market and industry.

In this presentation, we'll explain why CI/CD pipelines are the most dangerous potential attack surface of your software supply chain. To do this, we'll discuss the sorts of technologies we frequently encounter, how they're used, and why they are the most highly privileged and valuable targets in your company's entire infrastructure. We'll then discuss specific examples (with demos!) of novel abuses of intended functionality in automated pipelines which allow us to turn the build pipelines from a simple developer utility into Remote Code Execution-as-a-Service.

Is code-signing leading your team into a false sense of security while you programmatically build someone else's malware? Is it true that "any sufficiently advanced attacker is indistinguishable from one of your developers"? Have we critically compromised nearly every CI/CD pipeline we've ever touched? The answer to all of these questions is yes.

Fortunately, this presentation will not only teach you exactly how we did it and the common weaknesses we see in these environments, but also share key defensive takeaways that you can immediately apply to your own development environments.

[Talk Recording here](https://www.youtube.com/watch?v=Pe9nJLZvABM)