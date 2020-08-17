# Preparation

If you want to follow along or do the hands-on part in your own time, after the
tutorial, you will need to prepare a few things. Here's what.

## Kubernetes cluster

To follow along with the practical examples in this tutorial you'll need a
Kubernetes cluster that you can experiment with. Since at times you will be
deploying insecure code, please don't use your production cluster! 

You can run a cluster locally on your laptop, for example, we will be
using [Kubernetes IN Docker](https://kind.sigs.k8s.io) or `kind` for short
along with Helm to install and run apps on the cluster.

### Install kind

You can skip this step if you already have an up-to-date installation of `kind`.

On MacOS, using Homebrew:

```
brew install kind
```

On MacOS/Linux:

```
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.8.1/kind-$(uname)-amd64
chmod +x ./kind
mv ./kind /some-dir-in-your-PATH/kind
```

On Windows, using Chocolatey:

```
choco install kind
```

For more details see the [kind quickstart guide](https://kind.sigs.k8s.io/docs/user/quick-start/).

### Create cluster

To create a `kind` cluster (effectively a bunch of Docker containers running 
locally):

```
kind create cluster
```

Once it's up and running, check that you can see the worker node is up and running:

```
kubectl get nodes
```

This should show something like this:

```
NAME                 STATUS   ROLES    AGE   VERSION
kind-control-plane   Ready    master   78m   v1.18.2
```

Great! You have a Kubernetes cluster running locally that you can experiment with.
Where appropriate, we will create dedicated `kind` clusters with certain configurations.

## Install Helm

If you don't already have Helm on your laptop, you'll want to install that too.
Find full instructions in the [Helm documentation](https://helm.sh/docs/intro/install/) or here is a quick guide:

On MacOS, using Homebrew:

```
brew install helm
```

On MacOS/Linux:

```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod +x get_helm.sh
./get_helm.sh
```

On Windows, using Chocolatey:

```
choco install kubernetes-helm
```

!!! note
    If you have a fresh Kind installation there won't be any Helm charts installed
    yet, so a `helm ls` will return an empty list:

    ```
    $ helm ls
    NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
    ```

## Clone repo

In certain sections, you'll be using certain configuration files and Kubernetes
manifests as defined in the [`res/` directory](https://github.com/k8s-sec/cloud-native-security-tutorial/tree/master/res)
of our GitHub repository.

So, either download the content of said repository or simply clone the repo with:

```
git clone https://github.com/k8s-sec/cloud-native-security-tutorial.git
```

Unless we say otherwise, we assume `cloud-native-security-tutorial/` to be the
base directory, going forward.

OK and with this you're all set and ready to go!
