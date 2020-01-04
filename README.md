
# kindol

a [kind](https://github.com/kubernetes-sigs/kind) helper

Features:
* Install nginx-ingress with one command
* Dynamic enable/disable hostPort on docker host (expose ingress as localhost:80)
* Optional permanent hostPort mapping with `kindol create test --publish=80:80`(causes conflicts, use dynamic)

See demo and usage:

## demo
```
$ kindol create test
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:

Creating cluster "kindol-test" ...
 ✓ Ensuring node image (kindest/node:v1.16.3) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-kindol-test"
You can now use your cluster with:

kubectl cluster-info --context kind-kindol-test

Have a question, bug, or feature request? Let us know! https://kind.sigs.k8s.io/#community 🙂

$ kindol list
CONTAINER ID        IMAGE                  COMMAND                  CREATED              STATUS              PORTS                       NAMES
614205d2708d        kindest/node:v1.16.3   "/usr/local/bin/entr…"   About a minute ago   Up About a minute   127.0.0.1:51217->6443/tcp   kindol-test-control-plane

$ kindol ingress:enable test
installing with values:
controller:
  kind: DaemonSet
  daemonset:
    useHostPort: true
    hostPorts:
      http: 80
      https: 443
  service:
    enabled: false

NAME: ingress
LAST DEPLOYED: Sat Jan  4 16:07:56 2020
...<clipped helm output>

$ kindol kubectl test get pod
NAME                                                     READY   STATUS              RESTARTS   AGE
ingress-nginx-ingress-controller-j5sjx                   0/1     ContainerCreating   0          31s
ingress-nginx-ingress-default-backend-7d66f58f5f-5pt7m   1/1     Running             0          31s

$ kindol proxy:enable test
starting proxy of kindol-test-control-plane with --publish 127.0.0.1:80:80 --publish 127.0.0.1:443:44

$ curl localhost
default backend - 404

$ kindol proxy:disable test
proxy disabled

$ curl localhost
curl: (7) Failed to connect to localhost port 80: Connection refused
```

## install
```
git clone https://github.com/matti/kindol
cd kindol
ln -s $(pwd)/bin/kindol /usr/local/bin
```

## usage
```
USAGE: kindol COMMAND [ARG] [--opt=value]

list
  lists all kind clusters

create NAME [--publish=]
  creates a kind cluster

  use --publish=80:80 to permanently pubish port 80 from the control-plane
    (see proxy:enable for dynamic publish)

delete NAME
  deletes a kind cluster

ingress:enable NAME [--http=80 --https=443]
  installs ingress-nginx with helm as DaemonSet using hostPorts

ingress:disable NAME
  uninstalls ingress-nginx

proxy:enable NAME [--publish=127.0.0.1:80:80 --publish=127.0.0.1:443:443]
  publishes ports from the control-plane to the host

proxy:disable NAME
  disables the proxy

proxy:show
  shows the current proxy if enabled

helm NAME <helm args>
  use helm with the correct kubeconfig

kubectl|k NAME <kubectl args>
  use kubectl with the correct kubeconfig

version|--version
  prints kindol version

usage|help
  this text
```
