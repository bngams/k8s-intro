# Exercice 3 : Services ClusterIP et NodePort avec Minikube

Cet exercice vous permettra de comprendre comment exposer des applications dans Kubernetes en utilisant les services **ClusterIP** et **NodePort**. Nous allons travailler directement avec des Pods sans utiliser de Déploiements, et l'environnement est configuré avec **Minikube**.

## Prérequis

- **Minikube** doit être installé et en cours d'exécution.
- **kubectl** doit être configuré pour pointer vers le cluster Minikube.

Vérifiez que Minikube fonctionne correctement avec :

```bash
minikube status
```

Vous pouvez travailler dans un nouveau dossier `exercice2-services/`.

## 1. Création des Pods

Nous allons d'abord créer deux Pods : un serveur **frontend** et un serveur **backend**.

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
    - name: hello
      image: "gcr.io/google-samples/hello-go-gke:1.0"
      ports:
        - name: http
          containerPort: 80
```

Appliquez le Pod :

```bash
kubectl apply -f backend-pod.yaml
```

Vérifiez que le Pod fonctionne :

```bash
kubectl get pods
```

### b. Pod Frontend

Créez un fichier `frontend-pod.yaml` :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend-pod
  labels:
    app: hello
    tier: frontend
spec:
  containers:
  - name: nginx
    image: "gcr.io/google-samples/hello-frontend:1.0"
    ports:
      - containerPort: 80
```

Appliquez le Pod :

```bash
kubectl apply -f frontend-pod.yaml
```

Vérifiez que le Pod fonctionne :

```bash
kubectl get pods
```

## 2. Création des Services

### a. Service pour le Backend (ClusterIP)

Nous allons créer un service de type **ClusterIP** pour le backend.

Créez un fichier `backend-service.yaml` :

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello
spec:
  selector:
    app: backend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

Appliquez le service :

```bash
kubectl apply -f backend-service.yaml
```

Vérifiez le service :

```bash
kubectl get services
```

### b. Service pour le Frontend (NodePort)

Créez un fichier `frontend-service.yaml` :

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  selector:
    app: hello
    tier: frontend
  ports:
  - protocol: "TCP"
    port: 80
    targetPort: 80
    nodePort: 30080
  type: NodePort
```

Appliquez le service :

```bash
kubectl apply -f frontend-service.yaml
```

Vérifiez le service :

```bash
kubectl get services
```

## 3. Test de l'application

### a. Accéder au Frontend

Utilisez Minikube pour obtenir l'adresse IP du cluster :

```bash
minikube ip
```

Accédez à l'application via le navigateur ou avec `curl` :

```
http://<minikube-ip>:30080
```

Par exemple :

```bash
curl http://$(minikube ip):30080
```

### b. Vérifier les Logs

Vous pouvez consulter les logs des Pods pour vérifier leur fonctionnement :

```bash
kubectl logs backend-pod
kubectl logs frontend-pod
```

## 4. Nettoyage

Une fois l'exercice terminé, vous pouvez supprimer les ressources créées :

```bash
kubectl delete -f frontend-service.yaml
kubectl delete -f backend-service.yaml
kubectl delete -f frontend-pod.yaml
kubectl delete -f backend-pod.yaml
```

## Conclusion

Vous avez appris à créer des Pods et à les exposer via des services de type **ClusterIP** et **NodePort** dans un environnement Minikube. Cette méthode est utile pour comprendre les bases de l'exposition des applications dans Kubernetes.

