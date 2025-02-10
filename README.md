# Deploying Kong Air

This repo contains all the things you'll need to get started deploying Kong's demo system, Kong Air. 


### Tool dependencies

- [`helm`](https://helm.sh/docs/intro/install/)
- [`kind`](https://kind.sigs.k8s.io/)
- [`cloud-provider-kind`](https://github.com/kubernetes-sigs/cloud-provider-kind)
- [`kubectl`](https://kubernetes.io/docs/tasks/tools/)
- [`curl`](https://curl.se/download.html)
- [`jq`](https://jqlang.github.io/jq/download/)
- [`deck`](https://docs.konghq.com/deck/latest/)


## 1. Deploy a Kubernetes Cluster

You will need a Kubernetes cluster to deploy Kong Air. If you want to test locally, you can do so with `kind`.

1. Run `kind` to create a cluster:

```
kind create cluster --name kongair
```

2. You may need to untaint the control plane nodes to be able to run a load balancer (don't worry if this step fails, it's not always necessary):

```
kubectl label node kongair-control-plane node.kubernetes.io/exclude-from-external-load-balancers-
```

3. [Install](https://github.com/kubernetes-sigs/cloud-provider-kind) and run `cluster-provider-kind` (in another terminal) to provide load balancing:

```
cloud-provider-kind
```

## 2. Deploy the Kong Operator / KIC / Gateway

1. Install Gateway API CRDs:

```
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.0.0/standard-install.yaml
```

2. Install Kong Gateway Operator:

```
helm repo add kong https://charts.konghq.com
helm repo update kong
helm upgrade --install kgo kong/gateway-operator -n kong-system --create-namespace --set image.tag=1.4
kubectl -n kong-system wait --for=condition=Available=true --timeout=120s deployment/kgo-gateway-operator-controller-manager
```

3. Create Kong namespace:

```
kubectl apply -f gateway-setup/kong-ns.yaml
```

4. Create Gateway Configuration, Gateway Class, and Gateway:

```
kubectl apply -f gateway-setup/gateway-configuration.yaml -f gateway-setup/gateway-class.yaml -f gateway-setup/gateway.yaml
```

## 3. Build Images (optional)

This step is optional if the images are already pushed / available somewhere in a registry.

1. Build the following images and make them available to your cluster:
- github.com/KongAirlines/flights
- github.com/KongAirlines/bookings
- github.com/KongAirlines/customer
- github.com/KongAirlines/destinations

2. Make the images available to the cluster, for a kind cluster called `kongair` use the following:

```
kind load docker-image kongair/flights:latest kongair/destinations:latest kongair/customer:latest kongair/bookings:latest --name kongair
```


## 4. Deployment

Complete the required steps above then choose either K8s manifests or Helm install steps below:

### Kubernetes Manifests

1. Create application namespace:

```
kubectl apply -f k8s-raw/kong-air-ns.yaml
```

2. Apply application manifests:

```
kubectl apply -f  k8s-raw/bookings.yaml -f  k8s-raw/customers.yaml -f  k8s-raw/flights.yaml -f  k8s-raw/destinations.yaml
```

### Helm

1. Update relevant values in the `helm/kong-air/values.yaml` file then run the following:

```
helm install kong-air -f helm/kong-air/values.yaml ./helm/kong-air --namespace kong-air --create-namespace
```

## Test

1. Get the External IP of the Kong Dataplane Proxy and export it for ease of use:

```
kubectl get svc -n kong 
export KONG_SVC_IP=<proxy_external_ip>
```

2. Hit the Flights service:

```
curl $KONG_SVC_IP/flights
```

3. Hit the destinations service:

```
curl $KONG_SVC_IP/destinations
```

4. Hit the Bookings service:

```
curl $KONG_SVC_IP/bookings -H "x-consumer-username: jhamby"
```

5. Hit the Customers service:

```
curl $KONG_SVC_IP/customer -H "x-consumer-username: jhamby"
```
