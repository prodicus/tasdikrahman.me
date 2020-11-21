---
layout: post
title: "A few things to consider before choosing a big cluster or multiple smaller kubernetes clusters"
description: "A few things to consider before choosing a big cluster or multiple smaller kubernetes clusters"
tags: [kubernetes]
comments: true
share: true
cover_image: '/content/images/2020/04/'
---

This post is a continuation on the discussion which I was having with [@vineeth](https://twitter.com/VineethReddy02)

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">But why would someone choose one large cluster over multiple small clusters? Aren&#39;t multiple clusters already a pattern in enterprises?</p>&mdash; Vineeth Pothulapati (@VineethReddy02) <a href="https://twitter.com/VineethReddy02/status/1329656769900003329?ref_src=twsrc%5Etfw">November 20, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Context is when I came across a tweet which demonstrated the ability of kubernetes to scale uptill 15k nodes due to recent improvements.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">15k nodes cluster 🤯 <a href="https://t.co/VMWI7HeYHH">https://t.co/VMWI7HeYHH</a></p>&mdash; Tasdik Rahman (@tasdikrahman) <a href="https://twitter.com/tasdikrahman/status/1329615066405093379?ref_src=twsrc%5Etfw">November 20, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

The discussion was originally was around costs and how much would it take to run one such large kubernetes cluster, but it went into a different direction altogether. While this post is not a recommendation on what one should do, but the idea is to guide you to take a more informed decision with the data points and constraints that you have.

So what should the decision be? Rather, we can take the part where we discuss a few things about both sides of the coin

Before we start, big, here will be relative, but for the sake of this conversation, let's say you plan to run a cluster, with worker nodes greater than ~50 (this is a big cluster for me right now, more on the why part of it in a bit)

#### Multi tenancy

Hard multi-tenancy is something, which might not work the way as expected as of now on k8s, even with [netpols](https://kubernetes.io/docs/concepts/services-networking/network-policies/)/[namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)(that's what I have last checked, but please correct me here if I am wrong if have something which tells otherwise), which might make people concentrate towards having multiple k8s clusters to achieve it that way.

#### Upgrade charter

One operational aspect which might be important to note here is the burden of upgrades. Keeping up with upgrades, with the release cycle of kubernetes is not a trivial task. I have written about our experiences in this thread sometime back, to give you an idea of what really goes into one such upgrade.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">A few notes on <a href="https://twitter.com/kubernetes?ref_src=twsrc%5Etfw">@kubernetes</a> cluster upgrades on GKE (1/n)</p>&mdash; Tasdik Rahman (@tasdikrahman) <a href="https://twitter.com/tasdikrahman/status/1285619368353726465?ref_src=twsrc%5Etfw">July 21, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Even on a managed platform platform provided by the cloud vendors (where they maintain the control plane for you), it's still an operational heavy task, and we are not even talking about self managed clusters via for eg: [kops](https://github.com/kubernetes/kops/).

Multiply this effort with multiple such clusters, and you will end up needing to have people dedicated to just do this. There is no such thing as an LTS release as of now, unless you are fine with running super old kubernetes installations which would have CVE's reported and fixed in the upcoming releases/you are ok with doing big bang upgrades. Both of which might not be a great idea to begin with. Even if someone decides to run an archaic installation for long, if you feel running a super old installation will fly with your cloud provider, you will be in for a rude shock, where they can literally force upgrade your cluster(yes, they do it).

All of the toil which goes into cluster upgrades might have been a factor, which would have led to the initiation of this discussion [here](https://github.com/kubernetes/sig-release/issues/1290) where folks discuss about modifying the kubernetes release cadence. I for one [did vote](https://github.com/kubernetes/sig-release/issues/1290#issuecomment-709774428), on moving to 3 releases than 4 every year. But there are also arguments against doing slower releases, which might make the next release bloated with features and going against the philosophy of small incremental changes, but that discussion is for another post.

#### API deprecations

API's getting deprecated, for example [1.16](https://kubernetes.io/blog/2019/07/18/api-deprecations-in-1-16/) was a release which had a bunch of changes, which would have involved the cluster operators to upgrade their automation/helm charts to be compatible with those changes.

There's no one to blame here in case of API deprecations, an object getting stabler and getting promoted to a more stable API, is a natural progression. To enjoy the benefits of a stable kubernetes object, it only makes sense to move to the stable api rather than being stuck on one which is less stable/getting deprecated in the next release.

If you're a small org, with a small group of folks managing the k8s clusters, the manual toil will be quite high. The reasons are also obvious, the lack of bandwidht will attribute to them not being able to automate the redundant tasks required for the upgrade. Even if they manage to write some automation, the automation will become stale over time if not given prioritisation, as the domain changes. Plus, an average operations team will also have developer requests coming in their way, prioritising all this with a small task is just a hard task to begin with.

### References

- [https://kubernetes.io/docs/concepts/services-networking/network-policies/](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
- [https://blog.gojekengineering.com/how-we-upgrade-kubernetes-on-gke-91812978a055](https://blog.gojekengineering.com/how-we-upgrade-kubernetes-on-gke-91812978a055)
- [https://kubernetes.io/blog/2019/07/18/api-deprecations-in-1-16/](https://kubernetes.io/blog/2019/07/18/api-deprecations-in-1-16/)