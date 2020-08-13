# Policies 

One important tool in the defense in depth strategy are policies. These define
what is or is not allowed and part of it is usually an enforcement component.
In this section we will have a look at a number of different policies and how
you can apply them.

In the context of policies, the concept of least privileges is an important one,
so let's have a look at this for starters.

## Least privileges

With least privileges we mean to equip someone or something with exactly the
rights to carry out their or its task but not more. For example, if you consider
a program that needs to read from a specific location in the filesystem then
the operation (`read`) and the location (say, `/data`) would determine what
permissions are necessary. Conversely, said program would _not_ need `write` access
in addition and hence this would violate the least privileges principle.

First off we start with the simple case of a Kubernetes security context, 
allowing you to specify runtime policies around privileges and access control.

### Preparation

Let's create the cluster for it:

```
kind create cluster --name cnsectut \
     --config res/security-context-cluster-config.yaml
```

### Using a security context

We will be using an [example from the Kubernetes docs](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
so that you can read up on the details later on. 

First, launch the [pod with a security context](https://github.com/k8s-sec/cloud-native-security-tutorial/blob/master/res/pod-security-context.yaml) defined like so:

```
kubectl apply -f https://raw.githubusercontent.com/k8s-sec/cloud-native-security-tutorial/master/res/pod-security-context.yaml
```

Next, enter the pod:

```
kubectl exec -it security-context -- sh
```

And, in the pod, create a file as shown:

```
echo something > /data/content ; cd /data && id
```

What you see here is how the security context defined in the pod enforces file
and group ownership. But who enforces the definition of security contexts? That
is, how can you make sure that a developer creating a pod spec in fact thinks
of this? Enter [Pod Security
Policies](https://kubernetes.io/docs/concepts/policy/pod-security-policy/) or
PSP for short.


Learn more about least privileges practices in the container runtime context via:

- [rootlesscontaine.rs](https://rootlesscontaine.rs/)
- [canihaznonprivilegedcontainers.info](http://canihaznonprivilegedcontainers.info/)

Clean up with `kind delete cluster --name cnsectut` when you're done exploring
this topic.

## Network policies

So runtime policies are fun, but not the only thing that matters: next up, we
have a look at network policies. These kind of policies allow you to control
the communication patterns in-cluster (between pods) and concerning the outside
world (ingress and egress traffic):

![Kubernetes networking overview](img/kube-networking.png)

You might be surprised to learn that in Kubernetes by default all traffic
(in-cluster and to/from the outside world) is allowed. That is, any pod can see
and talk to any other pod by default as well as any connection to a pod running in your
Kubernetes cluster.

### Preparation

Let's create the cluster for the network policies walkthrough. 

The following can take a minute or two, depending on if you've pulled the
container images before or doing it the first time (then it can take 10min or more):

```
kind create cluster --name cnnp --config res/network-policy-cluster-config.yaml
```

Next, install the Calico controller and custom resources
(this is the CNI plugin that allows us to enforce the network policies):

```
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

Then, we need to patch the setup due to the fact we're using `kind` here 
(kudos to [Alex](https://alexbrand.dev/post/creating-a-kind-cluster-with-calico-networking/)
for the patch instructions):

```
kubectl -n kube-system set env daemonset/calico-node FELIX_IGNORELOOSERPF=true
```

Now you can verify the setup, and note that it can take some 5 min until 
you see all pods in the `Running` state:

```
kubectl -n kube-system get pods | grep calico-node
```

### Limit ingress traffic

Now let's create a workload and define communication paths:

```
#TBD: deploy pods in two namespaces that can talk to each other
#     and two that can not talk to each other (or ingress/egress)
```

Learn more about network policies via:

- [Exploring Network Policies in Kubernetes](https://banzaicloud.com/blog/network-policy/)
- [Best Practices for Kubernetes Network Policies](https://medium.com/@tufin/best-practices-for-kubernetes-network-policies-2b643c4b1aa)
- [Securing Kubernetes Cluster Networking](https://ahmet.im/blog/kubernetes-network-policy/)

Clean up with `kind delete cluster --name cnnp` when you're done exploring
this topic.

## OPA in action

The Open Policy Agent (OPA) project is a 

mini example via https://play.openpolicyagent.org/

OPA Gatekeeper


