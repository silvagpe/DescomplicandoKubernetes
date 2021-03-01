# Como fazer um Canary Deploy com o Kubernets?

## Criar um app

```app-v1.yml```


```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: giropops-v1
spec:
  replicas: 10
  selector:
    matchLabels:
      app: giropops
  template:
    metadata:
      labels:
        app: giropops
        version: "1.0.0"
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "32111"
    spec:
      containers:
      - name: giropops
        image: linuxtips/nginx-prometheus-exporter:1.0.0
        env:
        - name: VERSION
          value: "1.0.0"
        ports:
        - containerPort: 80
        - containerPort: 32111       
```

Criar o deployment da versão 1
```
kubectl create -f app-v1.yml
```

## Criar um service

```service-app.yml```

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: giropops
    run: nginx
    track: stable
  name: giropops
  namespace: default
spec:
  externalTrafficPolicy: Cluster
  ports:
  - nodePort: 32222
    name: http
    port: 80
    protocol: TCP
    targetPort: 80
  - nodePort: 32111
    name: prometheus
    port: 32111
    protocol: TCP
    targetPort: 32111
  selector:
    app: giropops
  sessionAffinity: None
  type: NodePort

```

Criar o services para o deployment da versão 1
```
kubectl create -f service-app.yml
```

## Criar o deploy canary 

```app-v2-canary.yml```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: giropops-v2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: giropops
  template:
    metadata:
      labels:
        app: giropops
        version: "2.0.0"
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "32111"
    spec:
      containers:
      - name: giropops
        image: linuxtips/nginx-prometheus-exporter:2.0.0
        env:
        - name: VERSION
          value: "2.0.0"
        ports:
        - containerPort: 80
        - containerPort: 32111
```
Criar um deployment com apenas uma replica para o deployment da versão 1
```
kubectl create -f app-v2-canary.yml

Mesmo só tendo publicado um service os deployments consegue se vincular ao service por meio da tag selector
```

## Aumentar a quantidade de replicas da versão 2

```app-v2.yml```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: giropops-v2
spec:
  selector:
    matchLabels:
      app: giropops
  replicas: 10
  template:
    metadata:
      labels:
        app: giropops
        version: "2.0.0"
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "32111"
    spec:
      containers:
      - name: giropops
        image: linuxtips/nginx-prometheus-exporter:2.0.0
        env:
        - name: VERSION
          value: "2.0.0"
        livenessProbe:  #São validações para saber quando o container estara pronto para receber requisições
          httpGet:
            path: /
            port: 80
            scheme: HTTP
        readinessProbe: #São validações para saber quando o container estara pronto para receber requisições
          httpGet:
            path: /
            port: 80
            scheme: HTTP
        ports:
        - containerPort: 80
        - containerPort: 32111
```

Para aplicar a alteração da versão usamos o comando apply
```
kubectl apply -f app-v2.yml

Nesse caso usamos o apply pq já existe um deploy rodando e queremo atualiza-lo
```

## Reduzir a quantidade de replicas da versão 1 antes de efetivar a exclusão

```app-v1.yml```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: giropops-v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: giropops
  template:
    metadata:
      labels:
        app: giropops
        version: "1.0.0"
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "32111"
    spec:
      containers:
      - name: giropops
        image: linuxtips/nginx-prometheus-exporter:1.0.0
        env:
        - name: VERSION
          value: "1.0.0"
        ports:
        - containerPort: 80
        - containerPort: 32111
```


Reduzindo a quantidade de replicas v1
```
kubectl get deployments
kubectl scale deployment --replicas=3 giropops-v1
...
...
...
kubectl scale deployment --replicas=1 giropops-v1
...
..
.
kubectl delete deployments giropops-v1


# Para ver a versões que foram "rodadas"
kubectl rollout history deployment giropops-v2

```

### Se a alteração foi bem sucessida então remover a versão 1
---


