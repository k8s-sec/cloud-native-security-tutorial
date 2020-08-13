# Policies 

One important tool in the defense in depth strategy are policies. These define
what is or is not allowed and part of it is usually an enforcement component.

In the context of policies, the concept of least privileges is an important one.
So let's have a look at this.

## Least privileges

First off we start with the simple case of a Kubernetes security context, 
allowing you to specify runtime policies around privileges and access control.

Let's create the cluster for it:

```
kind create cluster --name cnsectut --config res/security-context-cluster-config.yaml
```

We will be using an [example from the Kubernetes docs](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
so that you can read up on the details later on. 

```
kubectl apply -f https://raw.githubusercontent.com/k8s-sec/cloud-native-security-tutorial/master/res/pod-security-context.yaml

kubectl exec -it security-context -- sh

echo something > /data/content

cd /data && id

```

See more at http://canihaznonprivilegedcontainers.info/

## Network policies

Let's create the cluster for the network policies walkthrough 
(kudos to [Alex](https://alexbrand.dev/post/creating-a-kind-cluster-with-calico-networking/)
for the patch instructions):

```
# can take a minute or two, depending on if you've pulled the container images
# before or doing it the first time (then it can take 10min or more):
kind create cluster --name cnnp --config res/network-policy-cluster-config.yaml

# install Calico controller and custom resources, patch for testing:
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
kubectl -n kube-system set env daemonset/calico-node FELIX_IGNORELOOSERPF=true

# verify setup, can take some 5 min until you see all in the 'Running' state:
kubectl -n kube-system get pods | grep calico-node
```

Now let's create workloads and define communication paths:

```
#TBD: deploy pods in two namespaces that can talk to each other
#     and two that can not talk to each other (or ingress/egress)
```

See more at:

- [Exploring Network Policies in Kubernetes](https://banzaicloud.com/blog/network-policy/)
- [Best Practices for Kubernetes Network Policies](https://medium.com/@tufin/best-practices-for-kubernetes-network-policies-2b643c4b1aa)
- [Securing Kubernetes Cluster Networking](https://ahmet.im/blog/kubernetes-network-policy/)

## OPA in action

what is it

mini example via https://play.openpolicyagent.org/

OPA Gatekeeper


