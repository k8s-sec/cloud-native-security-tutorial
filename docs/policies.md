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

We have now seen container runtime policies as well as network policies in
action. You should by now have an idea what policies are and how to go about
defining and enforcing them. But did you notice one thing: for every type of 
policy, we had a different mechanism (and mind you, we only had a look at two 
types). Also, when you want to introduce further policies, for example, company
ones or maybe stuff you need to do to be compliant with some regulatory
framework such as PCI DSS. How do you deal with this in the context of
containers?

Meet the CNCF [Open Policy Agent](https://www.openpolicyagent.org/) (OPA) project.

OPA (pronounced "oh-pa") is a general-purpose policy engine that comes with a powerful rule-based
policy language called Rego (pronounced "ray-go"). Rego takes any kind of JSON
data as input and matches against a set of rules. It also comes with a long list
of [built-in](https://www.openpolicyagent.org/docs/latest/policy-reference/#built-in-functions)
functions, with from simple string manipulation stuff like
`strings.replace_n(patterns, string)` to fancy things such as
`crypto.x509.parse_certificates(string)`.

Enough theory, let's jump into the deep end using the [OPA Rego
playground](https://play.openpolicyagent.org/).

### OPA in action

Let's say you have the following input data, which is an array of timestamped
entries:

```
[
    {
        "msg": "within a week",
        "timestamp": "2020-03-27T12:00:00Z"
    },
    {
        "msg": "same year",
        "timestamp": "2020-10-09T12:00:00Z"
    },
    {
        "msg": "last year",
        "timestamp": "2019-12-24T12:00:00Z"
    }
]
```

So how can we check if a given entry is within a certain time window?
For example, you might require that a certain commit is not older than a week.

The following Rego file defines the policy we want to enforce (with a fixed
point in time `2020-04-01T12:00:00Z` as a reference):

```
package play

ts_reference := time.parse_rfc3339_ns("2020-04-01T12:00:00Z")

time_window_check[results] {
  some i
  msg := input[i].msg
  ts := time.parse_rfc3339_ns(input[i].timestamp)
  results := {
    "same_year" : same_year(ts),
    "within_a_week" : within_a_week(ts),
    "message": msg,
  }
}

same_year(ts) {
  [c_y, c_m, c_d] := time.date(ts_reference)
  [ts_y, ts_m, ts_d] := time.date(ts)
  ts_y == c_y
}

within_a_week(ts) {
  a_week := 60 * 60 * 24 * 7
  diff := ts_reference/1000/1000/1000 - ts/1000/1000/1000
  diff < a_week
  ts_reference > ts
}
```

Applying above Rego rule set to the input data, that is, querying for 
`time_window_check[results]` yields:

```
Found 1 result in 872.098 µs.
{
    "time_window_check": [
        {
            "message": "within a week",
            "same_year": true,
            "within_a_week": true
        }
    ],
    "ts_reference": 1585742400000000000
}
```

You can try this online yourself via the [prepared playground example](https://play.openpolicyagent.org/p/6v2EfFSq3l).

### OPA Gatekeeper

Now that you have an idea what OPA and Rego is you might wonder how hard it is
to use OPA/Rego in the context of Kubernetes. Turns out that writing, testing,
and enforcing these Rego rules is relatively hard and something that you don't
want to push onto individual folks. Good news is that the community came together 
to tackle this problem in the form of the [Gatekeeper](https://github.com/open-policy-agent/gatekeeper) project.

The Gatekeeper project solves the challenge of having to write and enforce Rego
rules by using the Kubernetes-native extension points of custom resources and
[dynamic admission control](https://kubernetes.io/blog/2019/03/21/a-guide-to-kubernetes-admission-controllers/).

As an end-user it's as simple as follows to use OPA with Gatekeeper. After
installing Gatekeeper, define and apply a custom resource like the following:

```
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: test-ns-label
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Namespace"]
  parameters:
    labels: ["test"]
```

This constraint above requires that all namespaces MUST have a label `test`.

But where is the Rego rule set I hear you ask?

Gatekeeper employs a separation of duties approach where (someone other than the
end-user) defines a template like so:

```
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels
        listKind: K8sRequiredLabelsList
        plural: k8srequiredlabels
        singular: k8srequiredlabels
      validation:
        # Schema for the `parameters` field
        openAPIV3Schema:
          properties:
            labels:
              type: array
              items: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlabels

        violation[{"msg": msg, "details": {"missing_labels": missing}}] {
          provided := {label | input.review.object.metadata.labels[label]}
          required := {label | label := input.parameters.labels[_]}
          missing := required - provided
          count(missing) > 0
          msg := sprintf("you must provide labels: %v", [missing])
        }
```

Above template effectively represents a custom resource definition that
Gatekeeper understands and can enforce via an Webhook registered in the
Kubernetes API server.

Learn more about OPA and Gatekeeper via:

- [Introducing Policy As Code: The Open Policy Agent (OPA)](https://www.cncf.io/blog/2020/08/13/introducing-policy-as-code-the-open-policy-agent-opa/)
- [OPA blog](https://blog.openpolicyagent.org/)
- [Styra Academy](https://academy.styra.com/)
- [OPA Gatekeeper: Policy and Governance for Kubernetes](https://kubernetes.io/blog/2019/08/06/opa-gatekeeper-policy-and-governance-for-kubernetes/)
- [CNCF webinar: Kubernetes with OPA Gatekeeper](https://www.youtube.com/watch?v=v4wJE3I8BYM)
