# Introduction

If you want to follow along with the hands-on examples in this tutorial, you'll
need a Kubernetes cluster and Helm to get started. You'll find instructions in
the [preparation](preparation.md) section.

Before we get to the hands-on part, let's review different ways an attacker might
compromise a Kubernetes cluster:

## Kubernetes attack vectors

![Kubernetes Attack Vectors](img/kuse_0101.png)

We won't have time to explore all of these in detail but we will be covering 
some of the most important topics.

- An attacker might take advantage of vulnerabilities in your application code.
  You can go a long way to address this by [scanning container images](scanning.md) for vulnerabilities.
- If an attacker gets a foothold within a container, you want to prevent themi
  from escaping the container to access the host or other components, by
  configuring container images with security in mind, and enforcing [policies](policies.md)
  for runtime and networking behavior.
- The Kubernetes components and APIs offer more potential surfaces for attack,
  so you want to check your [Kubernetes settings](settings.md) that might leave 
  your deployment vulnerable.
- At the root of many security issues lies human error, as well as bad actors. 
  You can limit human access to the cluster, making auditing easier, and enhancing security 
  by employing [GitOps](gitops.md).

With that, you had a look at the big picture, so let's move on to setting up
our hands-on environment.

