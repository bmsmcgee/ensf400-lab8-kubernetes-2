# ENSF 400 - Assignment 3 - Kubernetes

### Brandon McGee - 30125635

## Step 1: Start

Start Minikube and check the status to ensure it is running

```bash
$ minikube start
$ minikube status
```

The output should look like the following image

<img width="1593" alt="Screenshot 2024-04-03 at 23 18 47" src="https://github.com/bmsmcgee/ensf400-lab8-kubernetes-2/assets/98934884/2a4279bb-7afd-433c-b1bb-02e7b3ac71c7">

## Step 2: Enable Ingress

Enable the NGINX Ingress Controller through Minikube and check that it is enabled

_Input_
```bash
$ minikube addons enable ingress
$ minikube addons list
```

_Output_
<img width="1594" alt="Screenshot 2024-04-03 at 23 24 46" src="https://github.com/bmsmcgee/ensf400-lab8-kubernetes-2/assets/98934884/ac9fbe20-93cd-4ba7-abcc-a24d2c265ba3">

## Step 3A: Create

Create the ConfigMaps, Deployments, Services, Ingress, and Pods by running the following command to recursively 

_Input_
```bash
$ kubectl apply -f ./ -R
```

_Output_
<img width="1593" alt="Screenshot 2024-04-03 at 23 27 13" src="https://github.com/bmsmcgee/ensf400-lab8-kubernetes-2/assets/98934884/3df96570-cb47-46b9-9d6f-1db31e4ba10f">

## Step 3B: Checking
Skip to step 4 [HERE](#step-4-running-from-nginx-ingress).

Ensure the ConfigMaps, Deployments, Services, Ingress, and Pods are running and/or have been created properly using `get` and `describe`. Code for each is provided under output image.

### ConfigMaps
#### `nginx-configmaps.yaml`

_Input_
```bash
$ kubectl get configmaps
$ kubectl describe configmaps nginx-configmaps
```

_Output_
<img width="1596" alt="Screenshot 2024-04-03 at 23 30 24" src="https://github.com/bmsmcgee/ensf400-lab8-kubernetes-2/assets/98934884/f9f4dc05-2f43-4f87-8d63-6e4b01b6e428">

_File_
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-configmap
data:
  nginx.cfg: |
    upstream backend {
      server app-1:8080;
      server app-2:8080;
    }

    server {
      location / {
        proxy_pass http://backend;
      }
    }
```
---

### Deployments
#### `nginx-dep.yaml` 

_Input_
```bash
$ kubectl get deployments
$ kubectl describe deployments nginx-dep
```

_Output_
<img width="1595" alt="Screenshot 2024-04-03 at 23 36 17" src="https://github.com/bmsmcgee/ensf400-lab8-kubernetes-2/assets/98934884/209ebd20-b59d-4101-9916-d20c0b7c3f0d">

_File_
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-dep
  labels:
    app: nginx
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
        volumeMounts:
        - name: configmap-nginx
          mountPath: /etc/nginx/conf.d
      volumes:
      - name: configmap-nginx
        configMap: 
          name: nginx-configmap
          items:
          - key: nginx.cfg
            path: default.conf
```
---

#### `app-1-dep.yaml`

_Input_
```bash
$ kubectl get deployments
$ kubectl describe deployments app-1-dep
```

_Output_
<img width="1595" alt="Screenshot 2024-04-03 at 23 39 34" src="https://github.com/bmsmcgee/ensf400-lab8-kubernetes-2/assets/98934884/d86a1505-a9f6-451f-b50a-75238727643d">

_File_
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-1-dep
spec:
  selector:
    matchLabels:
      app: app-1
  template:
    metadata:
      labels:
        app: app-1
    spec:
      containers:
      - name: app-1
        image: ghcr.io/denoslab/ensf400-sample-app:v1
        imagePullPolicy: Always
        ports:
        - containerPort: 3000
```
---

#### `app-2-dep.yaml`

_Input_
```bash
$ kubectl get deployments
$ kubectl describe deployments app-2-dep
```

_Output_
<img width="1594" alt="Screenshot 2024-04-03 at 23 40 24" src="https://github.com/bmsmcgee/ensf400-lab8-kubernetes-2/assets/98934884/b7dd5016-b6e2-4c36-9966-80d460970f48">

_File_
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-2-dep
spec:
  selector:
    matchLabels:
      app: app-2
  template:
    metadata:
      labels:
        app: app-2
    spec:
      containers:
      - name: app-2
        image: ghcr.io/denoslab/ensf400-sample-app:v2
        imagePullPolicy: Always
        ports:
        - containerPort: 3000
```
---

### Services
#### `nginx-svc.yaml`

_Input_
```bash
$ kubectl get service
$ kubectl describe service nginx-svc
```

_Output_
<img width="1594" alt="Screenshot 2024-04-03 at 23 46 36" src="https://github.com/bmsmcgee/ensf400-lab8-kubernetes-2/assets/98934884/693367e2-bfea-4d1b-8ead-e5f097666de6">

_File_
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
```
---

#### `app-1-svc.yaml`

_Input_
```bash
$ kubectl get service
$ kubectl describe service app-1
```

_Output_
<img width="1592" alt="Screenshot 2024-04-03 at 23 49 32" src="https://github.com/bmsmcgee/ensf400-lab8-kubernetes-2/assets/98934884/7029a077-a303-4ce2-983a-46ad44adaf0f">

_File_
```yaml
apiVersion: v1
kind: Service
metadata:
  name: app-1
spec:
  selector:
    app: app-1
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 3000
```
---

#### `app-2-svc.yaml`

_Input_
```bash
$ kubectl get service
$ kubectl describe service app-2
```

_Output_
<img width="1591" alt="Screenshot 2024-04-03 at 23 50 47" src="https://github.com/bmsmcgee/ensf400-lab8-kubernetes-2/assets/98934884/be813dcd-0398-4619-8cc9-524613d0b315">

_File_
```yaml
apiVersion: v1
kind: Service
metadata:
  name: app-2
spec:
  selector:
    app: app-2
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 3000
```
---

### Ingress
#### `nginx-ingress.yaml`

_Input_
```bash
$ kubectl get ingress
$ kubectl describe ingress nginx-ingress
```

_Output_
<img width="1596" alt="Screenshot 2024-04-03 at 23 57 26" src="https://github.com/bmsmcgee/ensf400-lab8-kubernetes-2/assets/98934884/f0e38d9f-7a01-48c6-a3f2-ea1ba248fa91">

_File_
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  labels:
    app: nginx
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-svc
            port: 
              number: 80
```
---

#### `app-1-ingress.yaml`

_Input_
```bash
$ kubectl get ingress
$ kubectl describe ingress app-1-ingress
```

_Output_
<img width="1596" alt="Screenshot 2024-04-04 at 00 12 04" src="https://github.com/bmsmcgee/ensf400-lab8-kubernetes-2/assets/98934884/72390cc9-0925-4b10-8f98-e34ead2157b4">

_File_
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-1-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /app
        pathType: Prefix
        backend:
          service:
            name: app-1
            port: 
              number: 8080

```
---

#### `app-2-ingress.yaml`

_Input_
```bash
$ kubectl get ingress
$ kubectl describe ingress app-2-ingress
```

_Output_
<img width="1598" alt="Screenshot 2024-04-04 at 00 13 32" src="https://github.com/bmsmcgee/ensf400-lab8-kubernetes-2/assets/98934884/a17c1eea-63e6-494e-b7b7-cc166d6cb5ad">

_File_
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-2-ingress
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "30"
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /app
        pathType: Prefix
        backend:
          service:
            name: app-2
            port: 
              number: 8080

```
---

## Step 4: Running from `nginx-ingress`

Using an inline shell script, run the curl 20 times to get an output where the nginx is being used as a loadbalancer. The output of each app being selected is approximately equal.

_Input_
```bash
$ url="http://$(minikube ip)/"; count=0; for i in {1..20}; do
> curl "$url"
> echo ""
> if curl -s "$url" | grep -q "app-2"
> then ((count++))
> fi
> done; anticount=$((20 - count)); echo "app-1 appeared $anticount times and app-2 appeared $count times"
```

_Output_
<img width="1594" alt="Screenshot 2024-04-04 at 01 19 22" src="https://github.com/bmsmcgee/ensf400-lab8-kubernetes-2/assets/98934884/36e99196-1dd6-4813-8d5e-a88b0a6680d6">

## Step 5: Running the Ingress

Using inline shell script, run the curl 20 times to get an output where ingress is being used to select app-2 (the canary) 30% of the time, resulting in a 70/30 split of app-1/app-2.

_Input_
```bash
$ url="http://$(minikube ip)/app"; count=0; for i in {1..20}; do
> curl "$url"
> echo ""
> if curl -s "$url" | grep -q "app-2"
> then ((count++))
> fi
> done; anticount=$((20 - count)); echo "app-1 appeared $anticount times and app-2 appeared $count times"
```

_Output_
<img width="1596" alt="Screenshot 2024-04-04 at 01 24 09" src="https://github.com/bmsmcgee/ensf400-lab8-kubernetes-2/assets/98934884/c654198e-14d3-4e47-9a3e-f08ecbebb322">

## Step 6: Clean-Up

Run the following to delete all resources, then stop minikube, and close codespace.

_Input_
```bash
$ kubectl delete -f ./ -R
$ minikube stop
```

_Output_
<img width="1592" alt="Screenshot 2024-04-04 at 01 26 50" src="https://github.com/bmsmcgee/ensf400-lab8-kubernetes-2/assets/98934884/45405697-0c43-41c7-aa7d-b420ae5ac6cb">

