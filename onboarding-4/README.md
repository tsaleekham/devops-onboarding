# Run exercise

### Login Azure

```bash
az login  
```

### Build image

```bash
docker build -t devopsonboarding.azurecr.io/${NAME}:${VERSION} . 
```
for M1
```bash
docker buildx build --platform=linux/amd64 -t devopsonboarding.azurecr.io/${NAME}:${VERSION} . 
```

### Push image to Azure ACR
```bash
az acr login --name devopsonboarding
docker push devopsonboarding.azurecr.io/${NAME}:${VERSION}
```


## Create deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{NAME}}
spec:
  replicas: 3
  selector:
    matchLabels:
      app: {{NAME}}
  template:
    metadata:
      labels:
        app: {{NAME}}
    spec:
      containers:
      - name: {{NAME}}
        image: devopsonboarding.azurecr.io/{{NAME}}:latest
        ports:
        - containerPort: {{PORT}}
```

## Create service
```bash
kubectl expose deployment ${NAME} --type=NodePort --port=3000
```

## Create ingress controller
Run only for first time after created new cluster
```bash
kubectl apply -f nginx.yaml
kubectl apply -f nginx-policy.yaml
kubectl apply -f configmap.yaml
```

## Create ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{NAME}}
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - host: {{HOST}}
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: {{SERVICE_NAME}}
                port:
                  number: {{SERVICE_PORT}}
```