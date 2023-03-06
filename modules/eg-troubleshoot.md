# Lesson -  Troubleshooting Egress Gateway

## Identify the issue

It is essential to identify the nature of the issue before troubleshooting egress gateways. Common issues related to egress gateways are:

01. Egress gateway pods fail to deploy
02. Traffic does not get NATed to the egress gateway IPPOOL
03. Connections to the destination service/host fail



## Egress Gateway Deployment

Ensure the image pull secrets are copied to the correct namespace if the egress gateway pod deployment fails with an `ImagePullBackOff`. Copy the secret to the namespace where the egress gateway is to be deployed. 

```
kubectl get secret tigera-pull-secret --namespace=calico-system -o yaml | \
   grep -v '^[[:space:]]*namespace:[[:space:]]*calico-system' | \
   kubectl apply --namespace=[NAMESPACE] -f -
```

## natOutgoing on the egress IPPool

For most egress gateway scenarios, you should have `natOutgoing: false` on the egress IPPool.  If instead you have `natOutgoing: true` the egress gateway will SNAT to its IP - which is the intended egress gateway IP - but then the egress gateway’s node will also SNAT to its own IP - i.e. the node IP - thereby immediately overriding the egress gateway IP.

## Calico Enterprise License

If outgoing traffic is not NATed to an IP from the egress gateway IPPOOL, the Calico Enterprise license could be expired. Issue the below command to confirm that the `licensekey` is still valid. 

```
kubectl get licensekey -o yaml | grep expiry
```