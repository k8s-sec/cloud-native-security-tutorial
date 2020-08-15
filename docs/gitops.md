# GitOps

We wrap up this tutorial with a special topic insofar that it doesn't
demonstrate an attack of shows a control in action but rather discusses a
good practice. [GitOps](https://www.gitops.tech/) is a continuous or sometimes
called progressive deployment method. The source of truth for the state of the
deployments is Git and the way how a deployment is done is as follows:

1. As a developer or release engineer, you commit a change (for example, via
   a pull request in GitHub. 
1. A combination of bots and human reviewers comment on the commit, request
   changes and/or merge it, eventually.
1. In the Kubernetes cluster runs an agent that watches the Git repo and on
   changes, kicks off a new deployment.

In this setup, other than for read-only or potentially troubleshooting access
(with tight RBAC settings) the end-user does not have access to the Kubernetes
cluster. In other words, a `kubectl apply -f ...` is not possible, every change
of the application configuration is reviewed and part of an immutable log, the
Git repo's commit log. This allows at any point in time to reset the state to
a well-defined and good, previous state. Further, since it's formally and 
automatically on record who requested and who approved a change, auditing is
straightforward.

There are a number of tools available for applying GitOps in your team, for 
example:

- CNCF [Flux](https://docs.fluxcd.io)
- CNCF [ArgoCD](https://argoproj.github.io/argo-cd/)


To see GitOps in action, head over to the GitOps Toolkit and do go through
the [Get Started](https://toolkit.fluxcd.io/get-started/) guide.

Learn more about GitOps via:

- [Adopting GitOps for Kubernetes on AWS](https://acloudguru.com/blog/engineering/adopting-gitops-for-kubernetes-on-aws)
- Introduction To GitOps Toolkit: [video](https://www.youtube.com/watch?v=qQBtSkgl7tI) 
  and [slide deck](https://www.slideshare.net/weaveworks/gitops-toolkit-cloud-native-nordics-tech-talk).
