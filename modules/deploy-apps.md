# Lesson -  Deploy Applications

## Introduction

In this lab, you will deploy applications that'll be used to demonstrate the features of Calico egress gateway. 

## Deploy App1 and App2

```
kubectl apply -f -<<EOF
apiVersion: v1
kind: Namespace
metadata:
  name: apps
---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1
  namespace: apps
  labels:
    app: app1
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
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
---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: app2
  namespace: apps
  labels:
    app: app2
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app2
  template:
    metadata:
      labels:
        app: app2
    spec:
      containers:
      - name: app2
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
EOF
```

### Check Application Status

```
kubectl get pods -n apps


tigera@bastion:~$ kubectl get pods -n apps
NAME                    READY   STATUS    RESTARTS   AGE
app1-786b5b9b7f-fgw6h   1/1     Running   0          3m36s
app1-786b5b9b7f-rdgdg   1/1     Running   0          3m36s
app2-77c4f547ff-5744g   1/1     Running   0          2m49s
app2-77c4f547ff-cts6r   1/1     Running   0          2m49s
```

## Summary

The following lessons will explore egress gateway use cases using the apps. 
