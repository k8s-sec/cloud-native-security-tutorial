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

Last but not least we install [Ambassador](https://www.getambassador.io/) as an
ingress controller so that we can access to workloads from outside of the cluster:

```
kubectl apply -f https://github.com/datawire/ambassador-operator/releases/latest/download/ambassador-operator-crds.yaml && \
kubectl apply -n ambassador -f https://github.com/datawire/ambassador-operator/releases/latest/download/ambassador-operator-kind.yaml && \
kubectl wait --timeout=180s -n ambassador --for=condition=deployed ambassadorinstallations/ambassador
```

And with that we're ready to apply some network policies.

### Limit ingress traffic

Now let's see network policies in action by creating a public-facing workload
and define the communication paths.

First off, we want to do all of the following in a dedicated namespace called
`npdemo`:

```
kubectl create ns npdemo
```

Now, create the workload (deployment, service, ingress):

```
kubectl -n npdemo apply -f res/np-workload.yaml
```

When you now query the endpoint defined by the [ingress resource](https://github.com/k8s-sec/cloud-native-security-tutorial/blob/b5efb0ee54fbb3ded4f5cac8d7834f1e59881948/res/np-workload.yaml#L33)
you deployed in the previous step you will find that it works as expected
(remember: by default, all is allowed/open):

```
curl localhost/api
```

Now we shut down all traffic with:

```
kubectl -n npdemo apply -f res/deny-all.yaml
```

And try again:

```
curl localhost/api
```

As we'd have hoped and expected the access is now denied (might need to give it
a second or so until the change is picked up).

But now, how do we allow traffic to the frontend (represented by the NGINX web
server)? Well, we define another network policy that allows ingress to stuff
labelled with `role=frontend`:

```
kubectl -n npdemo apply -f res/allow-frontend.yaml && \
kubectl -n npdemo label pods --selector=app=nginx role=frontend
```

And now it should work again:

```
curl localhost/api
```

Learn more about network policies via:

- [Exploring Network Policies in Kubernetes](https://banzaicloud.com/blog/network-policy/)
- [Best Practices for Kubernetes Network Policies](https://medium.com/@tufin/best-practices-for-kubernetes-network-policies-2b643c4b1aa)
- [Securing Kubernetes Cluster Networking](https://ahmet.im/blog/kubernetes-network-policy/)

Clean up with `kind delete cluster --name cnnp` when you're done exploring
this topic.

## General purpose policies

The Open Policy Agent (OPA) project is a 

### OPA in action
mini example via https://play.openpolicyagent.org/

### OPA Gatekeeper

Learn more about OPA via:
- [Introducing Policy As Code: The Open Policy Agent (OPA)](https://www.cncf.io/blog/2020/08/13/introducing-policy-as-code-the-open-policy-agent-opa/)
