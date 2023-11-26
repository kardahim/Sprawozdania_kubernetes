# Stworzenie przestrzeni nazw

```bash
kubectl create namespace lab5
```

# Stworzenie quoty

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  creationTimestamp: null
  name: lab5-quota
  namespace: lab5
spec:
  hard:
    cpu: 2000m
    memory: 1.5Gi
    pods: "10"
status: {}
```

```bash
kubectl apply -f lab5-quota.yaml
```

![lab5_quota](./../assets/lab5_quota.png)

# Stworzenie poda

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: worker
  name: worker
  namespace: lab5
spec:
  containers:
    - image: nginx
      name: worker
      resources:
        limits:
          memory: "200Mi"
          cpu: "200m"
        requests:
          memory: "100Mi"
          cpu: "100m"
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
kubectl apply -f worker.yaml
```

![lab5_worker_pod](./../assets/lab5_pod_worker.png)

# Stworzenie deployment oraz service PHP-Apache

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
  namespace: lab5
spec:
  selector:
    matchLabels:
      run: php-apache
  template:
    metadata:
      labels:
        run: php-apache
    spec:
      containers:
        - name: php-apache
          image: registry.k8s.io/hpa-example
          ports:
            - containerPort: 80
          resources:
            limits:
              memory: 250Mi
              cpu: 250m
            requests:
              memory: 150Mi
              cpu: 150m
---
apiVersion: v1
kind: Service
metadata:
  name: php-apache
  namespace: lab5
  labels:
    run: php-apache
spec:
  ports:
    - port: 80
  selector:
    run: php-apache
```

```bash
kubectl apply -f php-apache.yaml
```

![lab_5_php_apache](../assets/lab5_php_apache.png)

# Użycie HPA

Maksymalna liczba replik jaka może być zastosowana w tej przestrzeni nazw to 5. Ponieważ gdy stworzylibyśmy 6 replik to w przypadku gdy worker oraz te repliki będą wykorzystywać maksymalne przydzielone zasoby to przekroczona zostanie dostępna liczba pamięci. Dalsze zwiększani/zmniejszanie liczby replik zależeć będzie od testów wydjaności.

![lab5_calc](./../assets/lab5_calc.png)

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
  namespace: lab5
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 5
  targetCPUUtilizationPercentage: 50
```

```bash
kubectl apply -f php-apache-autoscaler.yaml
```

![lab5_hpa](../assets/lab5_hpa.png)

# Weryfikacja obciążenia

Aby wygenerować ruch na podach:

```bash
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache.lab5.svc.cluster.local; done"
```

![lab5_tests_hp](./../assets/lab5_tests_hpa.png)

Maksymalne obciążenie wygenerowane przez skrypt to 700m/2 dla cpu. Dodatkowo maksymalnie 4 repliki zostały wygenerowane. Można zostawić powyższą konfigurację.
