# External Kong ingress controller

Helm chart to install ingress controller for kong outside the kubernetes cluster.

The official helm chart of `stable/kong` includes an ingress controller. However it installs a kong server by itself.
We probably have installed kong outside the kubernetes cluster, in this case we need to modify the ingress controller to make it work with the external kong server.

The modified kong kubernetes ingress controller can be found at https://github.com/trentzhou/kubernetes-ingress-controller/tree/external-kong. Right now it has not been accepted by the Kong community. The modified version adds an option `--kong-outside-kubernetes` to indicate that the kong service lives outside kubernetes.

Most part of the chart is just copied from [the official kong chart](https://github.com/helm/charts/tree/master/stable/kong). 

## How to use it

If you already have a kong server running, you can just use it. Otherwise you can deploy a new kong server. The simplest way to run kong is to use [the official docker-compose file](https://github.com/Kong/docker-kong/tree/master/compose). You can just clone the repo and run `docker-compose up -d`.

Then you can deploy the external kong ingress controller.

```
helm install --namespace=$NAMESPACE --name=$NAME --set kong.address=$KONG_ADDRESS chart/external-kong-ingress-controller
```


