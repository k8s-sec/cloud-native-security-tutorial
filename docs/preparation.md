# Preparation

## Create a Kubernetes cluster

To follow along with the practical examples in this tutorial you'll need a Kubernetes cluster that you can experiment with. Since at times you will be deploying insecure code, please don't use your production cluster! You can run a cluster locally on your laptop, for example using [Kind - Kubernetes IN Docker](https://kind.sigs.k8s.io). We'll also be using Helm to run software on the Kind cluster.

### Install kind

You can skip this step if you already have an up-to-date installation of `kind`.

On MacOS using Homebrew:

```
brew install kind
```

On MacOS / Linux:

```
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.8.1/kind-$(uname)-amd64
chmod +x ./kind
mv ./kind /some-dir-in-your-PATH/kind
```

On Windows using Chocolatey:

```
choco install kind
```

For more details see the [kind quickstart guide](https://kind.sigs.k8s.io/docs/user/quick-start/
).

### Create the kind cluster

```
kind create cluster
```

Once it's up and running, check that you can see the node is up and running:

```
kubectl get nodes
```

This should show something like this:

```
NAME                 STATUS   ROLES    AGE   VERSION
kind-control-plane   Ready    master   78m   v1.18.2
```

Great! You have a Kubernetes cluster running locally that you can experiment with.

## Install Helm

If you don't already have Helm on your laptop, you'll want to install that too. Find full instructions in the [Helm documentation](https://helm.sh/docs/intro/install/) or here is a quick guide:

On MacOS using Homebrew:

```
brew install helm
```

On MacOS / Linux:

```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod +x get_helm.sh
./get_helm.sh
```

On Windows using Chocolatey:

```
choco install kubernetes-helm
```

If you have a fresh Kind installation there won't be any Helm charts installed yet, so a `helm ls` will return an empty list:

```
$ helm ls
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
```