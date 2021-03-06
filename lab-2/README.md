# Lab 2 - Deploy Istio

Now that we have the Kubernetes cluster, we are ready to deploy Istio.

## Steps

* [1. Installing Istio](#1)
* [2. Seting up istioctl](#2)
* [3. Verify install](#3)
* [4. Configuring Add-ons](#4)

## <a name="1"></a> 1 - Installing Istio
As part of this lab we will install Istio 0.8.0 on your kubernetes cluster.

### Choose your own Adventure
Or your own **Adapters**...
A variety of Istio adapters and add-ons are available to enable out of the box. In this workshop, we will enable the Prometheus, ServiceGraph, Jaeger, Grafana and [SolarWinds](https://github.com/solarwinds/istio-adapter) adapters and add-ons. 

Configuration of the SolarWinds adapter is included as an optional lab, which enables shipping of metrics to [Appoptics](https://www.appoptics.com/), and/or logs to [Loggly](https://www.loggly.com/) and/or logs to [Papertrail](https://papertrailapp.com). To use the SolarWinds adapter, you may reserve your temporary, free account [here](https://docs.google.com/spreadsheets/d/1Rnqje4oQEQeaQRG24ApgdIzn8A2Pa3j5kzbm13_bJLA/edit). Proceed to [Optional Lab 2](optional.md) for configuration instructions.

```sh
curl https://raw.githubusercontent.com/leecalcote/istio-service-mesh-workshop/master/deployment_files/istio-0.8.0/istio-solarwinds-0.8.0.yaml | sed "s/<appoptics token>/$AOTOKEN/g" | sed "s/<loggly token>/$LOGGLY_TOKEN/g" > istio.yaml

kubectl apply -f istio.yaml
```

On `PWK` you will see an error message like this one:
```sh
  error: unable to recognize "istio.yaml": no matches for admissionregistration.k8s.io/, Kind=MutatingWebhookConfiguration
```

<img src="../img/info.png" width="48" align="left" /> We have Kubernetes version 1.8 on `PWK` which does have support for mutating webhooks which is the reason for the error. You can continue with the lab without any issues.


## <a name="2"></a> 2 - Verify install

Istio is deployed in a separate Kubernetes namespace `istio-system`. To check if Istio is deployed and also to see all the pieces that are deployed, we can do the following:

```sh
watch kubectl get all -n istio-system
```


## <a name="3"></a> 3 - Setting up istioctl
On a *nix system, you can setup istioctl by doing the following: 

```sh
curl -L https://git.io/getLatestIstio | sh -
```
The above command will get the latest Istio package, which at the time of this writing is 0.8.0.

```sh
export PATH="$PATH:/root/istio-0.8.0/bin"
```

To verify `istioctl` is setup lets try to print out the command help
```sh
istioctl version
```


## Configuring Add-ons

`Istio` comes with several addons like:
  1. [Prometheus](https://prometheus.io/)
  2. [Grafana](https://grafana.com/)
  3. [Zipkin](https://zipkin.io/)
  4. [Jaeger](https://www.jaegertracing.io/)
  5. [Service Graph](https://istio.io/docs/tasks/telemetry/servicegraph/)


For the folks who did NOT want to use Appoptics, you choose to use prometheus and grafana for viewing the metrics from `Istio`. 

For distributed tracing, you can choose between [Zipkin](https://zipkin.io/) or [Jaeger](https://www.jaegertracing.io/).

Service graph is another add-on which can be used to generate a graph of services within an Istio mesh. Service graph too is deployed as part of Istio in this lab.

Istio, deployed as part of this workshop, comes deployed with Prometheus, Grafana, Jaeger and Service Graph.

### Exposing services

By default, Istio addon services are deployed as `ClusterIP` type services, except Jaeger. We can expose the services outside the cluster by either changing the Kubernetes service type to NodePort or LoadBalancer or by port-forwarding or by configuring Kubernetes Ingress. In this lab, we will briefly demonstrate the NodePort and port-forwarding way of exposing services.

#### Exposing with NodePort
To expose them using NodePort service type, we can edit the services and change the service type from `ClusterIP` to `NodePort`

```sh
kubectl -n istio-system edit svc prometheus
```

```sh
kubectl -n istio-system edit svc grafana
```

```sh
kubectl -n istio-system edit svc servicegraph
```

Once this is done the services will be assigned dedicated ports on the hosts. 

To find the assigned ports for grafana:
```sh
kubectl -n istio-system get svc grafana
```

To find the assigned ports for prometheus:
```sh
kubectl -n istio-system get svc prometheus
```

To find the assigned ports for servicegraph:
```sh
kubectl -n istio-system get svc servicegraph
```

To find the assigned ports for jaeger, which was already exposed as a LoadBalancer service:
```sh
kubectl -n istio-system get svc tracing
```


#### Exposing with port-forward
To port-forward grafana:
```sh
kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=grafana \
  -o jsonpath='{.items[0].metadata.name}') 3000:3000 &
```

To port-forward prometheus:
```sh
kubectl -n istio-system port-forward \
  $(kubectl -n istio-system get pod -l app=prometheus -o jsonpath='{.items[0].metadata.name}') \
  9090:9090 &
```

To port-forward service graph:
```sh
kubectl -n istio-system port-forward \
  $(kubectl -n istio-system get pod -l app=servicegraph -o jsonpath='{.items[0].metadata.name}') \
  8088:8088 &
```

### Accessing exposed services

In `PWK`, once a port is exposed it will appear on top of the page as shown below as clickable hyperlinks:

![](img/exposed_ports.png)

We can click on the new relevant links now and navigate to the addons web UI, if available. 


If, for some reason, the links for the ports **DONOT** show up, you can grab the URL as shown in the image below, append the port and access the service.

![](img/expose_url.png)


Port-forwarding runs in the foreground. We have appeneded '&' to the end of the above 2 commands to run them in the background. If you donot want this behavior, please remove the '&'.




## [Continue to lab 3 - Deploy Sample Bookinfo app](../lab-3/README.md)
