# Policies 

One important tool in the defense in depth strategy are policies. These define
what is or is not allowed and part of it is usually an enforcement component.

In the context of policies, the concept of least privileges is an important one.
So let's have a look at this.

## Least privileges

First off we start with the simple case of a Kubernetes security context, 
allowing you to specify runtime policies around privileges and access control.

We will be using an [example from the Kubernetes docs](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
so that you can read up on the details later on. 

```
kubectl apply -f
https://raw.githubusercontent.com/k8s-sec/cloud-native-security-tutorial/master/res/pod-security-context.yaml
```

See more at http://canihaznonprivilegedcontainers.info/

## Network policies

https://banzaicloud.com/blog/network-policy/


https://medium.com/@tufin/best-practices-for-kubernetes-network-policies-2b643c4b1aa

## OPA in action

what is it

mini example via https://play.openpolicyagent.org/

OPA Gatekeeper


