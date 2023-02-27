# Lesson -  Egress Gateway per Deployment

## Introduction

This lab will demonstrate egress gateway per deployment; it includes the following tasks

1. Enable egress gateway support for namespaces and deployments
2. Deploy egress gateways 


### Deploy 


```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: netshoot
  namespace: app1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: netshoot
  template:
    metadata:
      annotations:
      labels:
        app: netshoot
    spec:
      containers:
        - name: netshoot
          image: nicolaka/netshoot:latest
          command: ["sleep", "3600"]
```

Remove the egress gateway annotation in the `app1` namespace applied in the previous lesson. 

```
kubectl annotate ns app1 egress.projectcalico.org/selector-
```

```
kubectl patch felixconfiguration.p default --type='merge' -p \
    '{"spec":{"egressIPSupport":"EnabledPerNamespaceOrPerPod"}}'
```

### Apply annotation to the app1 deployment

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1-deployment
  namespace: app1
  labels:
    app: app1
    tenant: tenant1
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app1
      tenant: tenant1
  template:
    metadata:
      annotations:
        egress.projectcalico.org/selector: 'egress-code == "red"'
      labels:
        app: app1
        tenant: tenant1
    spec:
      containers:
      - name: app1
        image: praqma/network-multitool
        env:
        - name: HTTP_PORT
          value: "1180"
        - name: HTTPS_PORT
          value: "11443"
        ports:
        - containerPort: 1180
          name: http-port
        - containerPort: 11443
          name: https-port
        resources:
          requests:
            cpu: "1m"
            memory: "20Mi"
          limits:
            cpu: "10m"
            memory: "20Mi"
```


## Summary

This lab demonstrated how to deploy Calico egress gateway and test its functionality by peering to an external BGP router. 



