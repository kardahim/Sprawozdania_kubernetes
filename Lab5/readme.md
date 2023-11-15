# Stworzenie przestrzeni nazw

```bash
kubectl create namespace lab5
```

# Stworzenie quoty

![lab5-quota](lab5-quota.yaml)

```bash
kubectl apply -f lab5-quota.yaml
```

![lab5_quota](./../assets/lab5_quota.png)

# Stworzenie poda

![worker](worker.yaml)

```bash
kubectl apply -f worker.yaml
```

![lab5_worker_pod](./../assets/lab5_pod_worker.png)

# Stworzenie deployment oraz service PHP-Apache

![php-apache](php-apache.yaml)

```bash
kubectl apply -f php-apache.yaml
```

![lab_5_php_apache](../assets/lab5_php_apache.png)

# Użycie HPA

Maksymalna liczba replik jaka może być zastosowana w tej przestrzeni nazw to 5. Ponieważ gdy stworzylibyśmy 6 replik to w przypadku gdy worker oraz te repliki będą wykorzystywać maksymalne przydzielone zasoby to przekroczona zostanie dostępna liczba pamięci. Dalsze zwiększani/zmniejszanie liczby replik zależeć będzie od testów wydjaności.

![lab5_calc](./../assets/lab5_calc.png)

![php-apache-autoscaler](php-apache-autoscaler.yaml)

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
