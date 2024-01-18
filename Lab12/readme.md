# Repozytorium DockerHub

Pierwszym krokiem jest stworzenie prywatego repozytorium oraz przesłanie na niego zbudowanego obrazu nginx z pliku dockerfile.

```dockerfile
FROM nginx:latest
```

```bash
docker login
docker build -t kardahim/lab12:lab12 .
docker push kardahim/lab12:lab12
```

![dockerhub](/assets/lab12_dockerhub_private_repo.png)

# Stworzenie secretu

Aby pobrać obraz do poda z prywatnego repozytorium należy podać token. Token będziemy przechowywać w sekrecie.

```bash
kubectl create secret docker-registry dockerhub-token-secret --docker-server=https://index.docker.io/v1/ --docker-username=kardahim --docker-password=SECRET_TOKEN
```

# Stworzenie poda

Gdy mamy już sekret należy stworzyć poda, który wykorzystuje prywatny obraz z platformy DockerHub.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test12
spec:
  containers:
    - name: test12-container
      image: kardahim/lab12:lab12
  imagePullSecrets:
    - name: dockerhub-token-secret
```

```bash
kubectl apply -f test12.yaml
```

![get pod](/assets/lab12_get_pod.png)

![descrie pod](/assets/lab12_describe_pod.png)
