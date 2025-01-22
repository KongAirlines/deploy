# Kind Setup

1. Install Metallb:

```
helm repo add metallb https://metallb.github.io/metallb
helm install metallb metallb/metallb --namespace metallb-system --create-namespace
kubectl -n metallb-system wait --for=condition=Available=true --timeout=120s deployment/metallb-controller
```

2. Run the following command to find the `kind` network IP CIDR:

```
export CIDR=$(docker network inspect -f '{{( index .IPAM.Config 1).Subnet}}' kind | cut -d ";" -f 1 | cut -d "." -f -2)
```

> ***Note***: You may need to switch the `.IPAM.Config` index to something other than `1` if the above command doesn't grab an IPv4 range.

3. Configure metal-lb and apply to the cluster:

```
envsubst < kind-setup/metallb-config.yaml | kubectl apply -f -
```