# Introduction

If you want to follow along with the hands-on examples in this tutorial, you'll need a Kubernetes cluster and Helm to get started. You'll find instructions in the [Preparation](preparation.md) page.

Before we get to the examples, let's review some of the different ways that an attacker might compromise a Kubernetes cluster.

## Kubernetes attack vectors

![Kubernetes Attack Vectors](img/kuse_0101.png)

We won't have time to explore all of these in detail but we will be covering some of the most important topics.

- An attacker might take advantage of vulnerabilities in your application code. You can go a long way to address this by [scanning container images for vulnerabilities](scanning.md)
- If an attacker gets a foothold within a container, you want to prevent them from escaping the container to access the host or other components, by [configuring container images with security in mind, and checking them with policies](policies.md)
- The Kubernetes components and APIs offer more potential surfaces for attack, so you want to [checking your Kubernetes configuration](settings.md) for settings that might leave your deployment vulnerable.
- At the root of many security issues lies human error, as well as bad actors. You can limit human access to the cluster, and thus [enhancing security using GitOps](gitops.md)

