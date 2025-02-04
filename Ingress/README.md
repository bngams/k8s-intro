# Exercice 4 : Utilisation des Ingress avec Minikube

Cet exercice vous permettra de comprendre comment utiliser des **Ingress** pour gérer l'accès à vos applications dans Kubernetes. Nous allons utiliser **Minikube** avec l'addon **Ingress** activé. Nous travaillerons directement avec des Pods et des Services.

## Prérequis

- **Minikube** doit être installé et en cours d'exécution.
- **kubectl** doit être configuré pour pointer vers le cluster Minikube.
- L'addon **Ingress** doit être activé.

### Activer l'addon Ingress

Assurez-vous que l'addon Ingress est activé :

```bash
minikube addons enable ingress
```

Vérifiez que l'Ingress Controller fonctionne :

```bash
kubectl get pods -n kube-system | grep ingress
```

## 1. Création des Pods

Nous allons créer deux applications simples : un **backend** et un **frontend**.

> [!CAUTION]
> /!\ Cela n'est pas précisé mais vous pouvez créer au préalable un Namespace auquel vous pouvez assigné tous les objets. Il sera d'autant plus simple de les supprimer à la fin de vos manipulations.

### a. Pod Backend

Créez un fichier `backend-pod.yaml` :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: backend-pod
  labels:
    app: backend
spec:
  containers:
  - name: backend
    image: mickaelbaron/microservice-backend:1.0
    ports:
    - containerPort: 8080
```

Appliquez le Pod :

```bash
kubectl apply -f backend-pod.yaml
```

### b. Pod Frontend

Créez un fichier `frontend-pod.yaml` :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend-pod
  labels:
    app: frontend
spec:
  containers:
  - name: frontend
    image: mickaelbaron/microservice-frontend:1.0
    ports:
    - containerPort: 8080
    env:
    - name: BACKEND_URL
      value: "http://backend-service:8080"
```

Appliquez le Pod :

```bash
kubectl apply -f frontend-pod.yaml
```

## 2. Création des Services

### a. Service pour le Backend (ClusterIP)

Créez un fichier `backend-service.yaml` :

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 8080
  type: ClusterIP
```

Appliquez le service :

```bash
kubectl apply -f backend-service.yaml
```

### b. Service pour le Frontend (ClusterIP)

Créez un fichier `frontend-service.yaml` :

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  selector:
    app: frontend
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 8080
  type: ClusterIP
```

Appliquez le service :

```bash
kubectl apply -f frontend-service.yaml
```

## 3. Création de l'Ingress

Créez un fichier `ingress.yaml` pour exposer les services via des chemins distincts :

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /frontend
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 8080
      - path: /backend
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 8080
```

Appliquez l'Ingress :

```bash
kubectl apply -f ingress.yaml
```

## 4. Configuration DNS locale

Ajoutez l'entrée suivante dans votre fichier `/etc/hosts` pour pointer `myapp.local` vers l'adresse IP de Minikube :

```bash
echo "$(minikube ip) myapp.local" | sudo tee -a /etc/hosts
```

> [!WARNING]
> Pour windows vous pouvez éditer le fichier C:\Windows\System32\drivers\etc

## 5. Test de l'application

### a. Accéder au Frontend

Accédez à l'application frontend :

```
http://myapp.local/frontend
```

### b. Accéder au Backend

Accédez à l'application backend :

```
http://myapp.local/backend
```

## 6. Vérification et Debugging

### a. Vérifier les Ingress

```bash
kubectl get ingress
```

### b. Vérifier les logs de l'Ingress Controller

```bash
kubectl logs -n kube-system $(kubectl get pods -n kube-system -l app.kubernetes.io/name=ingress-nginx -o jsonpath="{.items[0].metadata.name}")
```

## 7. Nettoyage

Une fois l'exercice terminé, vous pouvez supprimer les ressources créées :

```bash
kubectl delete -f ingress.yaml
kubectl delete -f frontend-service.yaml
kubectl delete -f backend-service.yaml
kubectl delete -f frontend-pod.yaml
kubectl delete -f backend-pod.yaml
```

## Conclusion

Vous avez appris à exposer des applications dans Kubernetes en utilisant des Ingress avec Minikube. Cette approche est essentielle pour gérer des accès HTTP plus complexes dans un cluster Kubernetes.

