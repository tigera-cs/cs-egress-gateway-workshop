# Lesson -  Egress Gateway per Deployment

## Introduction

This lab will demonstrate egress gateway per deployment; it includes the following tasks

1. Enable egress gateway support for namespaces and deployments
2. Create IP pools for `app1` and `app2` deployments
2. Deploy egress gateways 

![egw-intro](images/egw-per-deployment.png)

## Configure egress gateway

Remove the egress gateway annotation in the `apps` namespace applied in the previous lesson. 

```
kubectl annotate ns apps egress.projectcalico.org/selector-
```

### Enable egress gateway per namespace or per pod

```
kubectl patch felixconfiguration.p default --type='merge' -p \
    '{"spec":{"egressIPSupport":"EnabledPerNamespaceOrPerPod"}}'
```

### Create an `IPPool`

Create two IP pools. 

```
kubectl apply -f -<<EOF
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
  name: egress-ippool-app1
spec:
  cidr: 10.51.0.0/31
  blockSize: 31
  nodeSelector: "!all()"

---
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
  name: egress-ippool-app2
spec:
  cidr: 10.52.0.0/31
  blockSize: 31
  nodeSelector: "!all()"
EOF
```

### Deploy egress gateway for `app1` deployment

```
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: egress-gateway-app1
  namespace: apps 
  labels:
    egress-code: app1
spec:
  replicas: 1
  selector:
    matchLabels:
      egress-code: app1
  template:
    metadata:
      annotations:
        cni.projectcalico.org/ipv4pools: "[\"10.51.0.0/31\"]"
      labels:
        egress-code: app1
    spec:
      imagePullSecrets:
      - name: tigera-pull-secret
      nodeSelector:
        kubernetes.io/os: linux
      initContainers:
      - name: egress-gateway-init
        command: ["/init-gateway.sh"]
        image: quay.io/tigera/egress-gateway:v3.15.1
        env:
        # Use downward API to tell the pod its own IP address.
        - name: EGRESS_POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        securityContext:
          privileged: true
      containers:
      - name: egress-gateway
        command: ["/start-gateway.sh"]
        image: quay.io/tigera/egress-gateway:v3.15.1
        env:
        # Optional: comma-delimited list of IP addresses to send ICMP pings to; if all probes fail, the egress
        # gateway will report non-ready.
        - name: ICMP_PROBE_IPS
          value: ""
        # Only used if ICMP_PROBE_IPS is non-empty: interval to send probes.
        - name: ICMP_PROBE_INTERVAL
          value: "5s"
        # Only used if ICMP_PROBE_IPS is non-empty: timeout before reporting non-ready if there are no successful 
        # ICMP probes.
        - name: ICMP_PROBE_TIMEOUT
          value: "15s"
        # Optional comma-delimited list of HTTP URLs to send periodic probes to; if all probes fail, the egress
        # gateway will report non-ready.
        - name: HTTP_PROBE_URLS
          value: ""
        # Only used if HTTP_PROBE_URL is non-empty: interval to send probes.
        - name: HTTP_PROBE_INTERVAL
          value: "10s"
        # Only used if HTTP_PROBE_URL is non-empty: timeout before reporting non-ready if there are no successful 
        # HTTP probes.
        - name: HTTP_PROBE_TIMEOUT
          value: "30s"
        # Port that the egress gateway serves its health reports.  Must match the readiness probe and health
        # port defined below.
        - name: HEALTH_PORT
          value: "8080"
        - name: EGRESS_POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        securityContext:
          capabilities:
            add:
            - NET_ADMIN
        volumeMounts:
        - mountPath: /var/run
          name: policysync
        ports:
        - name: health
          containerPort: 8080
        readinessProbe:
          httpGet:
            path: /readiness
            port: 8080
          initialDelaySeconds: 3
          periodSeconds: 3
      terminationGracePeriodSeconds: 0
      volumes:
      - csi:
          driver: csi.tigera.io
        name: policysync
EOF
```

### Deploy egress gateway for `app2` deployment

```
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: egress-gateway-app2
  namespace: apps 
  labels:
    egress-code: app2
spec:
  replicas: 1
  selector:
    matchLabels:
      egress-code: app2
  template:
    metadata:
      annotations:
        cni.projectcalico.org/ipv4pools: "[\"10.52.0.0/31\"]"
      labels:
        egress-code: app2
    spec:
      imagePullSecrets:
      - name: tigera-pull-secret
      nodeSelector:
        kubernetes.io/os: linux
      initContainers:
      - name: egress-gateway-init
        command: ["/init-gateway.sh"]
        image: quay.io/tigera/egress-gateway:v3.15.1
        env:
        # Use downward API to tell the pod its own IP address.
        - name: EGRESS_POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        securityContext:
          privileged: true
      containers:
      - name: egress-gateway
        command: ["/start-gateway.sh"]
        image: quay.io/tigera/egress-gateway:v3.15.1
        env:
        # Optional: comma-delimited list of IP addresses to send ICMP pings to; if all probes fail, the egress
        # gateway will report non-ready.
        - name: ICMP_PROBE_IPS
          value: ""
        # Only used if ICMP_PROBE_IPS is non-empty: interval to send probes.
        - name: ICMP_PROBE_INTERVAL
          value: "5s"
        # Only used if ICMP_PROBE_IPS is non-empty: timeout before reporting non-ready if there are no successful 
        # ICMP probes.
        - name: ICMP_PROBE_TIMEOUT
          value: "15s"
        # Optional comma-delimited list of HTTP URLs to send periodic probes to; if all probes fail, the egress
        # gateway will report non-ready.
        - name: HTTP_PROBE_URLS
          value: ""
        # Only used if HTTP_PROBE_URL is non-empty: interval to send probes.
        - name: HTTP_PROBE_INTERVAL
          value: "10s"
        # Only used if HTTP_PROBE_URL is non-empty: timeout before reporting non-ready if there are no successful 
        # HTTP probes.
        - name: HTTP_PROBE_TIMEOUT
          value: "30s"
        # Port that the egress gateway serves its health reports.  Must match the readiness probe and health
        # port defined below.
        - name: HEALTH_PORT
          value: "8080"
        - name: EGRESS_POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        securityContext:
          capabilities:
            add:
            - NET_ADMIN
        volumeMounts:
        - mountPath: /var/run
          name: policysync
        ports:
        - name: health
          containerPort: 8080
        readinessProbe:
          httpGet:
            path: /readiness
            port: 8080
          initialDelaySeconds: 3
          periodSeconds: 3
      terminationGracePeriodSeconds: 0
      volumes:
      - csi:
          driver: csi.tigera.io
        name: policysync
EOF
```

### Apply annotation to the app1 deployment


```
kubectl patch deployment app1 -n apps -p '{"spec": {"template":{"metadata":{"annotations":{"egress.projectcalico.org/selector":"egress-code == \"app1\""}}}} }'
```

```
kubectl patch deployment app2 -n apps -p '{"spec": {"template":{"metadata":{"annotations":{"egress.projectcalico.org/selector":"egress-code == \"app2\""}}}} }'
```


## Configure BGP

### BGP configuration on Calico

Configure BGP to route traffic to the bastion host through the egress gateway.

```
kubectl apply -f -<<EOF
apiVersion: projectcalico.org/v3
kind: BGPPeer
metadata:
  name: bgppeer-global-64512
spec:
  peerIP: 10.0.1.10
  asNumber: 64512
---
apiVersion: projectcalico.org/v3
kind: BGPConfiguration
metadata:
  name: default
spec:
  serviceClusterIPs:
  - cidr: 10.49.0.0/16
  communities:
  - name: bgp-large-community
    value: 64512:120
  prefixAdvertisements:
  - cidr: 10.50.0.0/31
    communities:
    - bgp-large-community
    - 64512:120
  prefixAdvertisements:
  - cidr: 10.51.0.0/31
    communities:
    - bgp-large-community
    - 64512:120
  prefixAdvertisements:
  - cidr: 10.52.0.0/31
    communities:
    - bgp-large-community
    - 64512:120
EOF
```


## Summary

This lab demonstrated how to deploy Calico egress gateway and test its functionality by peering to an external BGP router. 



 kubectl patch deployment nginx -p '{"spec": {"template":{"metadata":{"annotations":{"sidecar.istio.io/inject":"false"}}}} }'

kubectl patch deployment app2 -n apps -p '{"spec": {"template":{"metadata":{"annotations":{"egress.projectcalico.org/selector":"egress-code == \"app2\""}}}} }'

kubectl patch deployment app2 -n apps -p '{"spec": {"template":{"metadata":{"annotations":{"egress.projectcalico.org/selector":'egress-code == "app2"'}}}} }'

APP2_POD=$(kubectl get pod -n apps --no-headers -o name | grep -i app2 | head -1) && echo $APP2_POD
kubectl exec -ti $APP2_POD -n apps -- sh
nc -zv 10.0.1.10 7777


