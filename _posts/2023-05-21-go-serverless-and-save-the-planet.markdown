---
title: "Go Serverless and save the planet"
date: 2023-05-21 08:08:00 +0000
description: Complete guide to self-host the Knative platform
categories:
 - blog
set: blog
order_number: 1
type: Document
layout: doc
toc: true
---

![Cover image]({{ site.baseUrl }}/assets/img/go-serverless-and-save-the-planet/cover.jpg "PHOTO: ILLUSTRATED FOR TERRAFORMATION BY JAMES YAMASAKI")
{:.tofigure}

With Serverless we could scale down to zero, and underlining hardware could be shared between many modules of our applications. Using hardware only when is needed for where is needed the most(with auto scaling) is a key to increase performance while reducing costs. So as result with less servers we do contribute to cooling down the planet. It's amazing to see that many young people are trying to save the planet from global warming. It seems a power transition is happening(e.g. in Europe) and we could expect a better decisions in continue &#129310;. Technologies also could help us in this path and let's hope we can go forward and become multiplanetary species even. What this post could do today is, to encourage people in software development to consider solutions which are more friendly to planet earth and help each other to adopt those solutions. Having said that some people think Trees are a faster solution to climate change, which frankly is simple and practical for getting into an action &#9996;.

### Cloud Native Computing Foundation (CNCF)

[landscape.cncf.io/serverless <i class="fa-solid fa-arrow-up-right-from-square"/>](https://landscape.cncf.io/serverless?fullscreen=yes&zoom=150){:target="_blank"}{:rel="noopener noreferrer"} is a great place to see what is happening with Serverless world and to compare projects based on stars, commit frequencies, etc. It worth your time to evaluate available options and comparing them before adopting them. And/or find a domain expert in the Serverless world and follow their advises. 

#### Knative platform
 In this post we are going to install [Knative platform <i class="fa-solid fa-arrow-up-right-from-square"/>](https://knative.dev/){:target="_blank"}{:rel="noopener noreferrer"} and config it for production use with minimum required hardware. Why Knative? for me it was [this post <i class="fa-solid fa-arrow-up-right-from-square"/>](https://www.salaboy.com/2021/11/30/my-story-with-knative/){:target="_blank"}{:rel="noopener noreferrer"} at first and after reading [Continuous Delivery for Kubernetes <i class="fa-solid fa-arrow-up-right-from-square"/>](http://mng.bz/jjKP){:target="_blank"}{:rel="noopener noreferrer"} book, I was sure Kubernetes and Knative need to be in my Serverless adventure's backpack. Plus I do live in a 3rd world country, which means no AWS, no easy access to ready to use public cloud providers. But renting a server(with below spec) could be as cheap as $8 a month. So some people are forced to go with self-host option and this post may help them.

## Complete guide to self-host the Knative platform

### Requirements

- [x] A server with a public IP(v4) address (e.g. 13.49.80.99)
    * 8 core CPU (6 core could work also)
    * 16G ram (6G could work also)
    * 30G disk (make sure you could add more storage if needed)
- [x] A domain name (e.g. example.com)

First make sure your server doesn't not have any limitation to access internet and docker registries, and if does maybe [this post <i class="fa-solid fa-arrow-up-right-from-square"/>]({% post_url 2023-05-12-use-cdn-to-bypass-ip-blocking %}){:target="_blank"}{:rel="noopener noreferrer"} could help you.


### Tested environment

- [x] Ubuntu 20.04 LTS
- [x] Microk8s 1.27/stable
- [x] Istio 1.17.2
- [x] Knative 1.10.1

### Required DNS Records

Following screenshot shows required type _A_ DNS Records.

![A DNS Records]({{ site.baseUrl }}/assets/img/go-serverless-and-save-the-planet/dns-records.png "A DNS Records (at cloudflare)")
{:.tofigure}


### Install Microk8s

Microk8s is known as a lightweight Kubernetes which is optimized to use minium possible resources and still offers what a normal Kubernetes could do. so it somehow matches with our overall goal to go Serverless with less servers.

{% highlight shell %}
sudo snap install microk8s --classic --channel=1.27/stable
{% endhighlight %}

At the moment version 1.27 is the latest stable version of Microk8s, you could check [here <i class="fa-solid fa-arrow-up-right-from-square"/>](https://microk8s.io/docs/setting-snap-channel){:target="_blank"}{:rel="noopener noreferrer"} to choose the right channel. And please note, different version, might have it's own differences in continue, and obviously you might face an issue, which we didn't.


{% highlight shell %}
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube
su - $USER
{% endhighlight %}

{% highlight shell %}
microk8s status --wait-ready
{% endhighlight %}

<details open><summary>output</summary>
{% highlight shell %}
microk8s is running
high-availability: no
  datastore master nodes: 127.0.0.1:19001
  datastore standby nodes: none
addons:
  enabled:
    dns                  # (core) CoreDNS
    ha-cluster           # (core) Configure high availability on the current node
    helm                 # (core) Helm - the package manager for Kubernetes
    helm3                # (core) Helm 3 - the package manager for Kubernetes
  disabled:
    cert-manager         # (core) Cloud native certificate management
    community            # (core) The community addons repository
    dashboard            # (core) The Kubernetes dashboard
    gpu                  # (core) Automatic enablement of Nvidia CUDA
    host-access          # (core) Allow Pods connecting to Host services smoothly
    hostpath-storage     # (core) Storage class; allocates storage from host directory
    ingress              # (core) Ingress controller for external access
    kube-ovn             # (core) An advanced network fabric for Kubernetes
    mayastor             # (core) OpenEBS MayaStor
    metallb              # (core) Loadbalancer for your Kubernetes cluster
    metrics-server       # (core) K8s Metrics Server for API access to service metrics
    minio                # (core) MinIO object storage
    observability        # (core) A lightweight observability stack for logs, traces and metrics
    prometheus           # (core) Prometheus operator for monitoring and logging
    rbac                 # (core) Role-Based Access Control for authorisation
    registry             # (core) Private image registry exposed on localhost:32000
    storage              # (core) Alias to hostpath-storage add-on, deprecated
{% endhighlight %}
</details>

#### Optional - disabling ha-cluster

Personally I think if you don't have a fast private network between your servers or you don't have a fast storage shared between your servers then Microk8s high availability feature might not work quite well as expected. In my case CPU usage was about 20% always busy. Unfortunately this decision is not something you could change later in your deployment, switch between ha-cluster and none ha-cluster will reset your cluster to initial state(removes everything!). In this setup we are going to have 1 server 1 node, and we prefer less cpu usage, so we did use following command to disable it.

{% highlight shell %}
microk8s disable ha-cluster --force
{% endhighlight %}

{% highlight shell %}
microk8s kubectl get all -A
{% endhighlight %}

<details open><summary>output</summary>
{% highlight shell %}
NAMESPACE   NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
default     service/kubernetes   ClusterIP   10.152.183.1   <none>        443/TCP   77s
{% endhighlight %}
</details>

##### Re-enable dns addons

{% highlight shell %}
microk8s enable dns
{% endhighlight %}

### Istio as ingress controller and network layer for Knative

In this guide we are going to use Istioctl to install Istio and we will have  _istio-system_ and _istio-ingress_ namespaces separated for having better security.

First we need to copy Microk8s config to default place for current user
{% highlight shell %}
mkdir -p ~/.kube
microk8s config > ~/.kube/config
{% endhighlight %}
Then add kubectl alias
{% highlight shell %}
alias kubectl='microk8s kubectl'
{% endhighlight %}
_~/.bashrc_
{: .right}

Or install kubectl by following command (if you want to use [krew <i class="fa-solid fa-arrow-up-right-from-square"/>](https://krew.sigs.k8s.io/){:target="_blank"}{:rel="noopener noreferrer"} plugins)
{% highlight shell %}
sudo snap install kubectl --classic
{% endhighlight %}
Then
{% highlight shell %}
mkdir ~/etc
cd ~/etc
curl -L https://istio.io/downloadIstio | sh -
cd ~/etc/istio-1.17.2/
export PATH=$PWD/bin:$PATH
istioctl install --set profile=minimal -y
{% endhighlight %}

Check installation
{% highlight shell %}
kubectl get all -n istio-system
{% endhighlight %}
<details open><summary>output</summary>
{% highlight shell %}
NAME                         READY   STATUS    RESTARTS   AGE
pod/istiod-57c965889-pdpv5   1/1     Running   0          28m

NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                                 AGE
service/istiod   ClusterIP   10.152.183.139   <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP   28m

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/istiod   1/1     1            1           28m

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/istiod-57c965889   1         1         1       28m

NAME                                         REFERENCE           TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/istiod   Deployment/istiod   <unknown>/80%   1         5         1          28m
{% endhighlight %}
</details>

#### Install istio-ingress with IstioOperator

{% highlight shell %}
istioctl operator init
{% endhighlight %}
<details open><summary>output</summary>
{% highlight shell %}
Installing operator controller in namespace: istio-operator using image: docker.io/istio/operator:1.17.2
Operator controller will watch namespaces: istio-system
✔ Istio operator installed
✔ Installation complete
{% endhighlight %}
</details>

<details open><summary>istio-ingress-operator.yaml</summary>
{% highlight yaml %}
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  name: ingress
  namespace: istio-system
spec:
  profile: empty # Do not install CRDs or the control plane
  components:
    ingressGateways:
    - name: istio-ingressgateway
      namespace: istio-ingress
      enabled: true
      label:
        # Set a unique label for the gateway. This is required to ensure Gateways
        # can select this workload
        istio: ingressgateway
  values:
    gateways:
      istio-ingressgateway:
        # Enable gateway injection
        injectionTemplate: gateway
{% endhighlight %}
</details>
_istio-ingress-operator.yaml_
{: .right}

{% highlight shell %}
kubectl create ns istio-ingress
kubectl apply -f istio-ingress-operator.yaml
{% endhighlight %}
Verify is ready
{% highlight shell %}
kubectl get all -n istio-ingress
{% endhighlight %}
<details open><summary>output</summary>
{% highlight shell %}
NAME                                        READY   STATUS    RESTARTS   AGE
pod/istio-ingressgateway-77b5b78896-bwvjw   1/1     Running   0          4m48s

NAME                           TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                                      AGE
service/istio-ingressgateway   LoadBalancer   10.152.183.50   <pending>     15021:30688/TCP,80:30792/TCP,443:32182/TCP   4m48s

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/istio-ingressgateway   1/1     1            1           4m48s

NAME                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/istio-ingressgateway-77b5b78896   1         1         1       4m48s

NAME                                                       REFERENCE                         TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/istio-ingressgateway   Deployment/istio-ingressgateway   <unknown>/80%   1         5         1          4m48s
{% endhighlight %}
</details>

#### Enable metallb addons
{% highlight shell %}
microk8s enable metallb:10.0.0.5-10.0.0.250
{% endhighlight %}
After successful installation, then your service must resolve external-ip from bare metal load balancer
{% highlight shell %}
kubectl get svc -n istio-ingress
{% endhighlight %}
<details open><summary>output</summary>
{% highlight shell %}
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                                      AGE
istio-ingressgateway   LoadBalancer   10.152.183.50   10.0.0.5      15021:30688/TCP,80:30792/TCP,443:32182/TCP   17m
{% endhighlight %}
</details>

#### Bind istio-ingressgateway to host port 80 and 443

{% highlight shell %}
kubectl edit deploy istio-ingressgateway -n istio-ingress
{% endhighlight %}

<details open><summary>istio-ingressgateway</summary>
{% highlight yaml %}
spec:
  template:
    spec:
      - env:
        ports:
        - containerPort: 15021
          protocol: TCP
        - containerPort: 8080
          hostPort: 80          # <- Add this line
          protocol: TCP
        - containerPort: 8443
          hostPort: 443         # <- Add this line too

{% endhighlight %}
</details>

Please note because of this binding, we can't scale more than one pod for a node, so in our case we set _replicas_ to 1. And for production environment with more thant one node, we could set _replicas_ to count of nodes and with a logic that each node should have one _istio-ingressgateway_ pod.

### Enable cert-manager addons

{% highlight shell %}
microk8s enable cert-manager
{% endhighlight %}
Verify is ready
{% highlight shell %}
kubectl get all -n cert-manager
{% endhighlight %}
<details open><summary>output</summary>
{% highlight shell %}
NAME                                           READY   STATUS    RESTARTS   AGE
pod/cert-manager-5d6bc46969-btqdd              1/1     Running   0          5m58s
pod/cert-manager-cainjector-7d8b8bb6b8-rjsx5   1/1     Running   0          5m58s
pod/cert-manager-webhook-5c5c5bb457-6zjcp      1/1     Running   0          5m58s

NAME                           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/cert-manager           ClusterIP   10.152.183.71    <none>        9402/TCP   5m58s
service/cert-manager-webhook   ClusterIP   10.152.183.177   <none>        443/TCP    5m58s

NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cert-manager              1/1     1            1           5m58s
deployment.apps/cert-manager-cainjector   1/1     1            1           5m58s
deployment.apps/cert-manager-webhook      1/1     1            1           5m58s

NAME                                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/cert-manager-5d6bc46969              1         1         1       5m58s
replicaset.apps/cert-manager-cainjector-7d8b8bb6b8   1         1         1       5m58s
replicaset.apps/cert-manager-webhook-5c5c5bb457      1         1         1       5m58s
{% endhighlight %}
</details>

#### Zerossl ClusterIssuer

<details open><summary>zerossl-cluster-issuer.yaml</summary>
{% highlight yaml %}
apiVersion: v1
kind: Secret
metadata:
  namespace: cert-manager
  name: zerossl-eab
stringData:
  secret: <CHANGE-TO-ACTUAL-SECRET>
---
apiVersion: v1
kind: Secret
metadata:
  namespace: cert-manager
  name: cloudflare-api-token
type: Opaque
stringData:
  api-token: <CHANGE-TO-ACTUAL-CLOUDFLARE-API-TOKEN>
---
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: zerossl-prod
spec:
  acme:
    # The ACME server URL
    server: https://acme.zerossl.com/v2/DV90
    externalAccountBinding:
      keyID: <CHANGE-TO-ACTUAL-KEY-ID>
      keySecretRef:
        name: zerossl-eab
        key: secret
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: zerossl-prod
    solvers:
    - dns01:
        cloudflare:
          apiTokenSecretRef:
            name: cloudflare-api-token
            key: api-token
{% endhighlight %}
</details>
_zerossl-cluster-issuer.yaml_
{: .right}

{% highlight shell %}
kubectl apply -f zerossl-cluster-issuer.yaml
{% endhighlight %}
Verify is ready
{% highlight shell %}
kubectl get ClusterIssuer
{% endhighlight %}
<details open><summary>output</summary>
{% highlight shell %}
NAME           READY   AGE
zerossl-prod   True    6s
{% endhighlight %}
</details>

#### Certificate for istio-ingress namespace

<details open><summary>certificate.yaml</summary>
{% highlight yaml %}
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  generation: 1
  name: star-example-com-tls
  namespace: istio-ingress
spec:
  dnsNames:
  - "example.com"
  - "*.example.com"
  - "*.s.example.com"
  - "*.default.s.example.com"
  issuerRef:
    group: cert-manager.io
    kind: ClusterIssuer
    name: zerossl-prod
  secretName: star-example-com-tls
  usages:
  - digital signature
  - key encipherment
{% endhighlight %}
</details>
_certificate.yaml_
{: .right}

{% highlight shell %}
kubectl apply -f certificate.yaml
{% endhighlight %}
Verify is ready
{% highlight shell %}
kubectl get Certificate -n istio-ingress
{% endhighlight %}
<details open><summary>output</summary>
{% highlight shell %}
NAME                     READY   SECRET                   AGE
star-example-com-tls   True    star-example-com-tls   7m38s
{% endhighlight %}
</details>

#### Istio IngressClass

<details open><summary>istio-ingress-class.yaml</summary>
{% highlight yaml %}
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: istio
spec:
  controller: istio.io/ingress-controller
{% endhighlight %}
</details>
_istio-ingress-class.yaml_
{: .right}


{% highlight shell %}
kubectl apply -f istio-ingress-class.yaml
{% endhighlight %}
Verify is ready
{% highlight shell %}
kubectl get IngressClass
{% endhighlight %}
<details open><summary>output</summary>
{% highlight shell %}
NAME    CONTROLLER                    PARAMETERS   AGE
istio   istio.io/ingress-controller   <none>       9s
{% endhighlight %}
</details>

#### Optional - NFS Persistent Volumes

In continue we are going to use NFS persistent volumes, and [this doc <i class="fa-solid fa-arrow-up-right-from-square"/>](https://microk8s.io/docs/nfs){:target="_blank"}{:rel="noopener noreferrer"} shows required steps and at the end you should be able to run following command and compare your output.

{% highlight shell %}
microk8s kubectl get storageclass
{% endhighlight %}

<details open><summary>output</summary>
{% highlight shell %}
NAME                PROVISIONER      RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
nfs-csi (default)   nfs.csi.k8s.io   Delete          Immediate           true                   72s
{% endhighlight %}
</details>

If your StorageClass is not mark as default, you could use following command.
{% highlight shell %}
microk8s kubectl patch storageclass nfs-csi -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
{% endhighlight %}

#### Optional - Enable registry addons
<details open><summary>output</summary>
{% highlight shell %}
microk8s enable registry
{% endhighlight %}
</details>

##### Registry ingress
<details open><summary>registry-ingress.yaml</summary>
{% highlight yaml %}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: registry
  namespace: container-registry
  annotations:
    kubernetes.io/ingress.class: istio
    cert-manager.io/cluster-issuer: "zerossl-prod"
spec:
  rules:
  - host: reg.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: registry
            port:
              number: 5000
  tls:
  - hosts:
    - reg.example.com
    secretName: star-example-com-tls
{% endhighlight %}
</details>
_registry-ingress.yaml_
{: .right}

{% highlight shell %}
kubectl apply -f registry-ingress.yaml
{% endhighlight %}
Verify is ready
{% highlight shell %}
kubectl get ingress -n container-registry
{% endhighlight %}
<details open><summary>output</summary>
{% highlight shell %}
NAME       CLASS    HOSTS               ADDRESS   PORTS     AGE
registry   <none>   reg.example.com             80, 443   5m39s
{% endhighlight %}
</details>
At this point you should be able to access your server from outside over https protocol
{% highlight shell %}
curl https://reg.example.com/v2/
{% endhighlight %}
<details open><summary>output</summary>
{% highlight json %}
{}
{% endhighlight %}
</details>

### Knative-serving installation

We are going to [install Knative by using Knative Operator <i class="fa-solid fa-arrow-up-right-from-square"/>](https://knative.dev/docs/install/operator/knative-with-operators/){:target="_blank"}{:rel="noopener noreferrer"} and latest version at the time is 1.10.1.

{% highlight shell %}
kubectl apply -f https://github.com/knative/operator/releases/download/knative-v1.10.1/operator.yaml
{% endhighlight %}

Verify is ready
{% highlight shell %}
kubectl get deployment knative-operator -n default
{% endhighlight %}
<details open><summary>output</summary>
{% highlight shell %}
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
knative-operator   1/1     1            1           12m
{% endhighlight %}
</details>

<details open><summary>knative-serving.yaml</summary>
{% highlight yaml %}
apiVersion: v1
kind: Namespace
metadata:
  name: knative-serving
---
apiVersion: operator.knative.dev/v1beta1
kind: KnativeServing
metadata:
  name: knative-serving
  namespace: knative-serving
spec:
  version: "1.10"
  ingress:
    istio:
      enabled: true
  config:
    domain:
      "s.example.com": ""
    network:
      auto-tls: "Enabled"
      autocreate-cluster-domain-claims: "true"
      namespace-wildcard-cert-selector: '{"matchExpressions": [{"key":"networking.knative.dev/enableWildcardCert", "operator": "In", "values":["true"]}]}'
    istio:
      gateway.knative-serving.knative-ingress-gateway: "istio-ingressgateway.istio-ingress.svc.cluster.local"
      local-gateway.knative-serving.knative-local-gateway: "knative-local-gateway.istio-ingress.svc.cluster.local"
    certmanager:
      issuerRef: |
        kind: ClusterIssuer
        name: zerossl-prod
{% endhighlight %}
</details>
_knative-serving.yaml_
{: .right}

{% highlight shell %}
kubectl apply -f knative-serving.yaml
{% endhighlight %}

Verify is ready
{% highlight shell %}
kubectl get KnativeServing knative-serving -n knative-serving
{% endhighlight %}
<details open><summary>output</summary>
{% highlight shell %}
NAME              VERSION   READY   REASON
knative-serving   1.10.1    True
{% endhighlight %}
</details>

#### Knative istio integration
{% highlight shell %}
kubectl apply -f https://github.com/knative/net-istio/releases/download/knative-v1.10.0/net-istio.yaml
{% endhighlight %}

<details open><summary>knative-peer-authentication.yaml</summary>
{% highlight yaml %}
apiVersion: v1
apiVersion: "security.istio.io/v1beta1"
kind: "PeerAuthentication"
metadata:
  name: "default"
  namespace: "knative-serving"
spec:
  mtls:
    mode: PERMISSIVE
{% endhighlight %}
</details>
_knative-peer-authentication.yaml_
{: .right}


{% highlight shell %}
kubectl apply -f knative-peer-authentication.yaml
{% endhighlight %}

#### Knative cert-manager integration

{% highlight shell %}
kubectl apply -f https://github.com/knative/net-certmanager/releases/download/knative-v1.10.0/release.yaml
{% endhighlight %}

Verify is ready
{% highlight shell %}
kubectl get deployment net-certmanager-controller -n knative-serving
{% endhighlight %}
<details open><summary>output</summary>
{% highlight shell %}
NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
net-certmanager-controller   1/1     1            1           95s
{% endhighlight %}
</details>

##### Enable wildcard Certificate for default namespace

{% highlight shell %}
kubectl label ns default networking.knative.dev/enableWildcardCert=true
{% endhighlight %}

#### Ensure configmaps updated successfully

{% highlight shell %}
kubectl get configmap config-istio -n knative-serving -o yaml
kubectl get configmap config-domain -n knative-serving -o yaml
kubectl get configmap config-network -n knative-serving -o yaml
kubectl get configmap config-certmanager -n knative-serving -o yaml
{% endhighlight %}

### HelloWorld service for test serving

<details open><summary>hello-service.yaml</summary>
{% highlight yaml %}
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: helloworld-go
  namespace: default
spec:
  template:
    spec:
      containers:
        - image: ghcr.io/knative/helloworld-go:latest
          env:
            - name: TARGET
              value: "Go Sample v1"
{% endhighlight %}
</details>
_hello-service.yaml_
{: .right}

{% highlight shell %}
kubectl apply -f hello-service.yaml
{% endhighlight %}

Verify is ready
{% highlight shell %}
kctl get ksvc -n default
{% endhighlight %}

<details open><summary>output</summary>
{% highlight shell %}
NAME            URL                                             LATESTCREATED         LATESTREADY           READY   REASON
helloworld-go   https://helloworld-go.default.s.example.com   helloworld-go-00001   helloworld-go-00001   True
{% endhighlight %}
</details>

Finally test helloworld service to be accessible from outside

{% highlight shell %}
curl https://helloworld-go.default.s.example.com
{% endhighlight %}

<details open><summary>output</summary>
{% highlight shell %}
Hello Go Sample v1!
{% endhighlight %}
</details>

### Config custom domain for helloworld-go servcie


<details open><summary>hello-domain-mapping.yaml</summary>
{% highlight yaml %}
apiVersion: serving.knative.dev/v1alpha1
kind: DomainMapping
metadata:
  name: hello.example.com
  namespace: default
spec:
  ref:
    name: helloworld-go
    kind: Service
    apiVersion: serving.knative.dev/v1
{% endhighlight %}
</details>
_hello-domain-mapping.yaml_
{: .right}

{% highlight shell %}
kubectl apply -f hello-domain-mapping.yaml
{% endhighlight %}

Verify is ready
{% highlight shell %}
kctl get domainmapping -n default
{% endhighlight %}

<details open><summary>output</summary>
{% highlight shell %}
NAME                  URL                           READY   REASON
hello.example.com   https://hello.example.com   True
{% endhighlight %}
</details>

{% highlight shell %}
curl https://hello.example.com
{% endhighlight %}

<details open><summary>output</summary>
{% highlight shell %}
Hello Go Sample v1!
{% endhighlight %}
</details>

That's all, in this post we did deploy a Knative service and make it securely available to our customers &#127867;.

{% highlight shell %}
kubectl get all -A
{% endhighlight %}

<details><summary>output</summary>
{% highlight shell %}
NAMESPACE            NAME                                             READY   STATUS    RESTARTS         AGE
cert-manager         pod/cert-manager-5d6bc46969-btqdd                1/1     Running   1 (5h51m ago)    24h
cert-manager         pod/cert-manager-cainjector-7d8b8bb6b8-rjsx5     1/1     Running   1 (5h50m ago)    24h
cert-manager         pod/cert-manager-webhook-5c5c5bb457-6zjcp        1/1     Running   0                24h
container-registry   pod/registry-9865b655c-ftngg                     1/1     Running   0                23h
default              pod/knative-operator-7b7d4bbc7d-pj48l            1/1     Running   0                22h
default              pod/operator-webhook-74d9489bf8-7vdhr            1/1     Running   0                22h
istio-ingress        pod/istio-ingressgateway-7f5958c7d9-l2qfc        1/1     Running   0                24h
istio-operator       pod/istio-operator-79d6df8f9d-m9zkw              1/1     Running   3 (5h51m ago)    25h
istio-system         pod/istiod-57c965889-pdpv5                       1/1     Running   0                29h
knative-serving      pod/activator-658c5747b-zzvjm                    1/1     Running   0                21h
knative-serving      pod/autoscaler-f989fbf86-fzszz                   1/1     Running   0                21h
knative-serving      pod/autoscaler-hpa-5d7668d747-gs6zs              1/1     Running   0                21h
knative-serving      pod/controller-64b9dcc975-h4vnq                  1/1     Running   0                21h
knative-serving      pod/domain-mapping-7b87f895b6-zxbkk              1/1     Running   0                21h
knative-serving      pod/domainmapping-webhook-54cddcb594-xxmgl       1/1     Running   0                21h
knative-serving      pod/net-certmanager-controller-575898d58-bv6x5   1/1     Running   0                20h
knative-serving      pod/net-certmanager-webhook-7cd899c855-pps7t     1/1     Running   0                20h
knative-serving      pod/net-istio-controller-84cb8b59fb-dxvgq        1/1     Running   0                21h
knative-serving      pod/net-istio-webhook-8d785b78d-jwqqf            1/1     Running   0                21h
knative-serving      pod/webhook-7698bcf68f-qxd7f                     1/1     Running   0                21h
kube-system          pod/coredns-7745f9f87f-lswgs                     1/1     Running   2 (28h ago)      47h
kube-system          pod/csi-nfs-controller-6f844cdc89-slhdt          3/3     Running   12 (5h51m ago)   30h
kube-system          pod/csi-nfs-node-r6h7c                           3/3     Running   6 (28h ago)      30h
kube-system          pod/hostpath-provisioner-58694c9f4b-ztmgq        1/1     Running   3 (5h51m ago)    23h
metallb-system       pod/controller-8467d88d69-sqlsf                  1/1     Running   0                25h
metallb-system       pod/speaker-j4b5r                                1/1     Running   0                25h

NAMESPACE            NAME                                  TYPE           CLUSTER-IP       EXTERNAL-IP                                             PORT(S)                                              AGE
cert-manager         service/cert-manager                  ClusterIP      10.152.183.71    <none>                                                  9402/TCP                                             24h
cert-manager         service/cert-manager-webhook          ClusterIP      10.152.183.177   <none>                                                  443/TCP                                              24h
container-registry   service/registry                      NodePort       10.152.183.166   <none>                                                  5000:32000/TCP                                       23h
default              service/helloworld-go                 ExternalName   <none>           knative-local-gateway.istio-ingress.svc.cluster.local   80/TCP                                               19h
default              service/helloworld-go-00001           ClusterIP      10.152.183.45    <none>                                                  80/TCP,443/TCP                                       19h
default              service/helloworld-go-00001-private   ClusterIP      10.152.183.68    <none>                                                  80/TCP,443/TCP,9090/TCP,9091/TCP,8022/TCP,8012/TCP   19h
default              service/kubernetes                    ClusterIP      10.152.183.1     <none>                                                  443/TCP                                              47h
default              service/operator-webhook              ClusterIP      10.152.183.107   <none>                                                  9090/TCP,8008/TCP,443/TCP                            22h
istio-ingress        service/istio-ingressgateway          LoadBalancer   10.152.183.50    10.0.0.5                                                15021:30688/TCP,80:30792/TCP,443:32182/TCP           25h
istio-ingress        service/knative-local-gateway         ClusterIP      10.152.183.228   <none>                                                  80/TCP                                               21h
istio-operator       service/istio-operator                ClusterIP      10.152.183.132   <none>                                                  8383/TCP                                             25h
istio-system         service/istiod                        ClusterIP      10.152.183.139   <none>                                                  15010/TCP,15012/TCP,443/TCP,15014/TCP                29h
istio-system         service/knative-local-gateway         ClusterIP      10.152.183.216   <none>                                                  80/TCP                                               21h
knative-serving      service/activator-service             ClusterIP      10.152.183.19    <none>                                                  9090/TCP,8008/TCP,80/TCP,81/TCP,443/TCP              21h
knative-serving      service/autoscaler                    ClusterIP      10.152.183.49    <none>                                                  9090/TCP,8008/TCP,8080/TCP                           21h
knative-serving      service/autoscaler-bucket-00-of-01    ClusterIP      10.152.183.147   <none>                                                  8080/TCP                                             21h
knative-serving      service/autoscaler-hpa                ClusterIP      10.152.183.156   <none>                                                  9090/TCP,8008/TCP                                    21h
knative-serving      service/controller                    ClusterIP      10.152.183.35    <none>                                                  9090/TCP,8008/TCP                                    21h
knative-serving      service/domainmapping-webhook         ClusterIP      10.152.183.238   <none>                                                  9090/TCP,8008/TCP,443/TCP                            21h
knative-serving      service/net-certmanager-controller    ClusterIP      10.152.183.104   <none>                                                  9090/TCP,8008/TCP                                    20h
knative-serving      service/net-certmanager-webhook       ClusterIP      10.152.183.227   <none>                                                  9090/TCP,8008/TCP,443/TCP                            20h
knative-serving      service/net-istio-webhook             ClusterIP      10.152.183.138   <none>                                                  9090/TCP,8008/TCP,443/TCP                            21h
knative-serving      service/webhook                       ClusterIP      10.152.183.150   <none>                                                  9090/TCP,8008/TCP,443/TCP                            21h
kube-system          service/kube-dns                      ClusterIP      10.152.183.10    <none>                                                  53/UDP,53/TCP,9153/TCP                               47h
metallb-system       service/webhook-service               ClusterIP      10.152.183.163   <none>                                                  443/TCP                                              25h

NAMESPACE        NAME                          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system      daemonset.apps/csi-nfs-node   1         1         1       1            1           kubernetes.io/os=linux   30h
metallb-system   daemonset.apps/speaker        1         1         1       1            1           kubernetes.io/os=linux   25h

NAMESPACE            NAME                                             READY   UP-TO-DATE   AVAILABLE   AGE
cert-manager         deployment.apps/cert-manager                     1/1     1            1           24h
cert-manager         deployment.apps/cert-manager-cainjector          1/1     1            1           24h
cert-manager         deployment.apps/cert-manager-webhook             1/1     1            1           24h
container-registry   deployment.apps/registry                         1/1     1            1           23h
default              deployment.apps/helloworld-go-00001-deployment   0/0     0            0           19h
default              deployment.apps/knative-operator                 1/1     1            1           22h
default              deployment.apps/operator-webhook                 1/1     1            1           22h
istio-ingress        deployment.apps/istio-ingressgateway             1/1     1            1           25h
istio-operator       deployment.apps/istio-operator                   1/1     1            1           25h
istio-system         deployment.apps/istiod                           1/1     1            1           29h
knative-serving      deployment.apps/activator                        1/1     1            1           21h
knative-serving      deployment.apps/autoscaler                       1/1     1            1           21h
knative-serving      deployment.apps/autoscaler-hpa                   1/1     1            1           21h
knative-serving      deployment.apps/controller                       1/1     1            1           21h
knative-serving      deployment.apps/domain-mapping                   1/1     1            1           21h
knative-serving      deployment.apps/domainmapping-webhook            1/1     1            1           21h
knative-serving      deployment.apps/net-certmanager-controller       1/1     1            1           20h
knative-serving      deployment.apps/net-certmanager-webhook          1/1     1            1           20h
knative-serving      deployment.apps/net-istio-controller             1/1     1            1           21h
knative-serving      deployment.apps/net-istio-webhook                1/1     1            1           21h
knative-serving      deployment.apps/webhook                          1/1     1            1           21h
kube-system          deployment.apps/coredns                          1/1     1            1           47h
kube-system          deployment.apps/csi-nfs-controller               1/1     1            1           30h
kube-system          deployment.apps/hostpath-provisioner             1/1     1            1           23h
metallb-system       deployment.apps/controller                       1/1     1            1           25h

NAMESPACE            NAME                                                        DESIRED   CURRENT   READY   AGE
cert-manager         replicaset.apps/cert-manager-5d6bc46969                     1         1         1       24h
cert-manager         replicaset.apps/cert-manager-cainjector-7d8b8bb6b8          1         1         1       24h
cert-manager         replicaset.apps/cert-manager-webhook-5c5c5bb457             1         1         1       24h
container-registry   replicaset.apps/registry-9865b655c                          1         1         1       23h
default              replicaset.apps/helloworld-go-00001-deployment-6466c46f55   0         0         0       19h
default              replicaset.apps/knative-operator-7b7d4bbc7d                 1         1         1       22h
default              replicaset.apps/operator-webhook-74d9489bf8                 1         1         1       22h
istio-ingress        replicaset.apps/istio-ingressgateway-77b5b78896             0         0         0       25h
istio-ingress        replicaset.apps/istio-ingressgateway-7f5958c7d9             1         1         1       25h
istio-operator       replicaset.apps/istio-operator-79d6df8f9d                   1         1         1       25h
istio-system         replicaset.apps/istiod-57c965889                            1         1         1       29h
knative-serving      replicaset.apps/activator-658c5747b                         1         1         1       21h
knative-serving      replicaset.apps/autoscaler-f989fbf86                        1         1         1       21h
knative-serving      replicaset.apps/autoscaler-hpa-5d7668d747                   1         1         1       21h
knative-serving      replicaset.apps/controller-64b9dcc975                       1         1         1       21h
knative-serving      replicaset.apps/domain-mapping-7b87f895b6                   1         1         1       21h
knative-serving      replicaset.apps/domainmapping-webhook-54cddcb594            1         1         1       21h
knative-serving      replicaset.apps/net-certmanager-controller-575898d58        1         1         1       20h
knative-serving      replicaset.apps/net-certmanager-webhook-7cd899c855          1         1         1       20h
knative-serving      replicaset.apps/net-istio-controller-84cb8b59fb             1         1         1       21h
knative-serving      replicaset.apps/net-istio-webhook-8d785b78d                 1         1         1       21h
knative-serving      replicaset.apps/webhook-7698bcf68f                          1         1         1       21h
kube-system          replicaset.apps/coredns-7745f9f87f                          1         1         1       47h
kube-system          replicaset.apps/csi-nfs-controller-6f844cdc89               1         1         1       30h
kube-system          replicaset.apps/hostpath-provisioner-58694c9f4b             1         1         1       23h
metallb-system       replicaset.apps/controller-8467d88d69                       1         1         1       25h

NAMESPACE         NAME                                                       REFERENCE                         TARGETS          MINPODS   MAXPODS   REPLICAS   AGE
istio-ingress     horizontalpodautoscaler.autoscaling/istio-ingressgateway   Deployment/istio-ingressgateway   <unknown>/80%    1         5         1          25h
istio-system      horizontalpodautoscaler.autoscaling/istiod                 Deployment/istiod                 <unknown>/80%    1         5         1          29h
knative-serving   horizontalpodautoscaler.autoscaling/activator              Deployment/activator              <unknown>/100%   1         20        1          21h
knative-serving   horizontalpodautoscaler.autoscaling/webhook                Deployment/webhook                <unknown>/100%   1         5         1          21h

NAMESPACE   NAME                                              LATESTCREATED         LATESTREADY           READY   REASON
default     configuration.serving.knative.dev/helloworld-go   helloworld-go-00001   helloworld-go-00001   True

NAMESPACE   NAME                                               CONFIG NAME     K8S SERVICE NAME   GENERATION   READY   REASON   ACTUAL REPLICAS   DESIRED REPLICAS
default     revision.serving.knative.dev/helloworld-go-00001   helloworld-go                      1            True             0                 0

NAMESPACE   NAME                                      URL                                             READY   REASON
default     route.serving.knative.dev/helloworld-go   https://helloworld-go.default.s.taakcloud.com   True

NAMESPACE   NAME                                        URL                                             LATESTCREATED         LATESTREADY           READY   REASON
default     service.serving.knative.dev/helloworld-go   https://helloworld-go.default.s.taakcloud.com   helloworld-go-00001   helloworld-go-00001   True

NAMESPACE   NAME                                                    URL                           READY   REASON
default     domainmapping.serving.knative.dev/hello.taakcloud.com   https://hello.taakcloud.com   True
{% endhighlight %}
</details>
