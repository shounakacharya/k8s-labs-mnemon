# Service Mesh with Istio
## Download Istio
1. Go to the [Istio release](https://github.com/istio/istio/releases/tag/1.19.0) page to download the installation file for your OS, or download and extract the latest release automatically (Linux or macOS). We would be using the version 1.19.0 for this lab purposes. In this lab, since we would be using Linux and a specific version, we would be directly passing in these infomration in the curl command directly as under:
```bash
vagrant@master-node:~$ curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.19.0 TARGET_ARCH=x86_64 sh -
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   102    0   102    0     0    292      0 --:--:-- --:--:-- --:--:--   293
100  4899  100  4899    0     0   4671      0  0:00:01  0:00:01 --:--:--  4671

Downloading istio-1.19.0 from https://github.com/istio/istio/releases/download/1.19.0/istio-1.19.0-linux-amd64.tar.gz ...

Istio 1.19.0 Download Complete!

Istio has been successfully downloaded into the istio-1.19.0 folder on your system.

Next Steps:
See https://istio.io/latest/docs/setup/install/ to add Istio to your Kubernetes cluster.

To configure the istioctl client tool for your workstation,
add the /home/vagrant/istio-1.19.0/bin directory to your environment path variable with:
         export PATH="$PATH:/home/vagrant/istio-1.19.0/bin"

Begin the Istio pre-installation check by running:
         istioctl x precheck

Need more information? Visit https://istio.io/latest/docs/setup/install/
vagrant@master-node:~$
```
2. Move to the Istio package directory:
```bash
vagrant@master-node:~$ cd istiod-1.19.0
vagrant@master-node:~/istio-1.19.0$
```

The installation directory contains:
    - Sample applications in `samples/`
    - The `istioctl` client binary in the `bin/` directory
3. Add the `istioctl` client to your path (Linux or MacOS)
```bash
vagrant@master-node:~/istio-1.19.0$export PATH=$PWD/bin:$PATH
```


## Install Istio
1. Istio comes with a lot of [configuration profiles](https://istio.io/latest/docs/setup/additional-setup/config-profiles/) which let you configure Istio with a lot of defaults for a particular use case. In this lab and subsequent examples, we would be using the `demo` profile which has a lot of configuration settings apt for getting started. Beyond the ones provided by Istio, platform vendors like AWS, Openshift etc. provide their own [platform specific configuration profiles](https://istio.io/latest/docs/setup/platform-setup/) which are ideal for running istio on these platforms 
```bash
vagrant@master-node:~/istio-1.19.0$ istioctl install --set profile=demo -y
✔ Istio core installed
✔ Istiod installed
✔ Egress gateways installed
✔ Ingress gateways installed
✔ Installation complete
Made this installation the default for injection and validation.
vagrant@master-node:~/istio-1.19.0$
```
2. Istio requires namespaces to be labeled where Istio can automatically inject sidecars. In this tutorial we will enable automatic envoy injection in the the default namespace

```bash
vagrant@master-node:~/istio-1.19.0$ kubectl label namespace default istio-injection=enabled
namespace/default labeled
vagrant@master-node:~/istio-1.19.0$
```

## Checking external IP and Ports for Istio Applications
Istio applications are deployed within a mesh and Istio takes care of the routing between various applications. However, Istio applications can also be exposed outside the cluster using the Istio Ingress Gateway component which is deployed during the istio installation. The following points show how to get the details of the Istio Ingress Gateway. Please note that this is not the same ingress gateway based upon nginx that we have discussed previously

1. Firstly for typing less, we create two environment variables holding our `istio-ingress-gateway` and the `istio-namespace` name

```bash
vagrant@master-node:~$ export INGRESS_NAME=istio-ingressgateway
vagrant@master-node:~$ echo $INGRESS_NAME
istio-ingressgateway
vagrant@master-node:~$ export INGRESS_NS=istio-system
vagrant@master-node:~$ echo $INGRESS_NS
istio-system
vagrant@master-node:~$
```
2. Next we create some more environment variables which points to the various NodePorts for each of the Istio Ingress Gateway service. These are ports for ingressing traffic into Istio Applications on TCP, HTTP and HTTPS

```bash
vagrant@master-node:~$ export INGRESS_PORT=$(kubectl -n "${INGRESS_NS}" get service "${INGRESS_NAME}" -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
vagrant@master-node:~$ export SECURE_INGRESS_PORT=$(kubectl -n "${INGRESS_NS}" get service "${INGRESS_NAME}" -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
vagrant@master-node:~$ export TCP_INGRESS_PORT=$(kubectl -n "${INGRESS_NS}" get service "${INGRESS_NAME}" -o jsonpath='{.spec.ports[?(@.name=="tcp")].nodePort}')
vagrant@master-node:~$ echo $INGRESS_PORT
31976
vagrant@master-node:~$ echo $SECURE_INGRESS_PORT
31109
vagrant@master-node:~$ echo $TCP_INGRESS_PORT
31928
vagrant@master-node:~$
```
3. Also set the Ingress Host environment variable to the IP of the first worker node. Technically 

