# Raw Kubernetes Install Instructions

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
kubectl apply -f kong-ns.yaml
```

4. Create Gateway Configuration, Gateway Class, and Gateway:

```
kubectl apply -f gateway-configuration.yaml -f gateway-class.yaml -f gateway.yaml
```

5. Create application namespace:

```
kubectl apply -f kong-air-ns.yaml
```

6. Apply application manifests:

```
kubectl apply -f bookings.yaml -f customers.yaml -f flights.yaml -f routes.yaml
```