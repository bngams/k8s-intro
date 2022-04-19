# Création d'un Pod

## Exercice

Dans cet exercice, vous allez créer une spécification pour lancer un premier Pod.

### 1. Création de la spécification

Créez un fichier yaml *whoami.yaml* définissant un Pod ayant les propriétés suivantes:
- nom du Pod: *whoami*
- image du container: *containous/whoami*
- nom du container: *whoami*

### 2. Lancement du Pod

Lancez le Pod à l'aide de *kubectl*

### 3. Vérification

Listez les Pods lancés et assurez vous que le Pod *whoami* apparait bien dans cette liste.

### 4. Details du Pod

Observez les détails du Pod à l'aide de *kubectl* et retrouvez l'information de l'image utilisée par le container *whoami*.

### 5. Accès à l'application via un port-forward

Avec la commande *kubectl port-forward* envoyer une requête à l'application

### 6. Suppression du Pod

Supprimez le Pod.

---

## Correction

### 1. Création de la spécification

La spécification, définie dans le fichier *whoami.yaml*, est la suivante:

```
apiVersion: v1             
kind: Pod                  
metadata:
  name: whoami
spec:
  containers:
  - name: whoami
    image: containous/whoami
```

### 2. Lancement du Pod

Le Pod peut être créé avec la commande suivante:

```
$ kubectl apply -f whoami.yaml
```

### 3. Vérification

La commande suivante permet de lister les Pods présent:

```
$ kubectl get pods
NAME      READY     STATUS    RESTARTS   AGE
whoami    1/1       Running   0          14s
```

Note: il est aussi possible de précisez *pod* (au singulier) ou simplement *po*

```
$ kubectl get pod
NAME      READY     STATUS    RESTARTS   AGE
whoami    1/1       Running   0          16s

$ kubectl get po
NAME      READY     STATUS    RESTARTS   AGE
whoami    1/1       Running   0          22s
```

### 4. Details du Pod

Les details d'un Pod peuvent être obtenus avec la commande suivante:

```
$ kubectl describe pod whoami
Name:         whoami
Namespace:    default
Priority:     0
Node:         workers-3ce65/10.131.116.83
Start Time:   Mon, 18 May 2020 16:21:03 +0200
Labels:       <none>
Annotations:  Status:  Running
IP:           10.244.1.4
IPs:
  IP:  10.244.1.4
Containers:
  whoami:
    Container ID:   docker://c9072713a7367015e8e77fe81853bc90d68b07a308227144812270fd4e2e1deb
    Image:          containous/whoami
    Image ID:       docker-pullable://containous/whoami@sha256:7d6a3c8f91470a23ef380320609ee6e69ac68d20bc804f3a1c6065fb56cfa34e
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Mon, 18 May 2020 16:21:08 +0200
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-njqbg (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-njqbg:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-njqbg
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age        From                    Message
  ----    ------     ----       ----                    -------
  Normal  Scheduled  <unknown>  default-scheduler       Successfully assigned default/whoami to workers-3ce65
  Normal  Pulling    44s        kubelet, workers-3ce65  Pulling image "containous/whoami"
  Normal  Pulled     41s        kubelet, workers-3ce65  Successfully pulled image "containous/whoami"
  Normal  Created    41s        kubelet, workers-3ce65  Created container whoami
  Normal  Started    41s        kubelet, workers-3ce65  Started container whoami
```

Note: les commandes suivantes peuvent également être utilisées:
- kubectl describe pods whoami
- kubectl describe po whoami
- kubectl describe pods/whoami
- kubectl describe pod/whoami
- kubectl describe po/whoami

Dans cette sortie, on peut voir la liste des containers du Pods et l'image utilisée pour le container *whoami*.

Il est également possible d'obtenir la spécification du Pod avec la commande suivante dans laquelle on spécifie via *-o yaml* le format de sortie.

```
$ kubectl get po/whoami -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"whoami","namespace":"default"},"spec":{"containers":[{"image":"containous/whoami","name":"whoami"}]}}
  creationTimestamp: "2020-05-18T14:21:03Z"
  name: whoami
  namespace: default
  resourceVersion: "76052"
  selfLink: /api/v1/namespaces/default/pods/whoami
  uid: 7faccd41-bf0f-4d2e-9627-d0efe1e03116
spec:
  containers:
  - image: containous/whoami
    imagePullPolicy: Always
    name: whoami
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-njqbg
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: workers-3ce65
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: default-token-njqbg
    secret:
      defaultMode: 420
      secretName: default-token-njqbg
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2020-05-18T14:21:03Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2020-05-18T14:21:08Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2020-05-18T14:21:08Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2020-05-18T14:21:03Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://c9072713a7367015e8e77fe81853bc90d68b07a308227144812270fd4e2e1deb
    image: containous/whoami:latest
    imageID: docker-pullable://containous/whoami@sha256:7d6a3c8f91470a23ef380320609ee6e69ac68d20bc804f3a1c6065fb56cfa34e
    lastState: {}
    name: whoami
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2020-05-18T14:21:08Z"
  hostIP: 10.131.116.83
  phase: Running
  podIP: 10.244.1.4
  podIPs:
  - ip: 10.244.1.4
  qosClass: BestEffort
  startTime: "2020-05-18T14:21:03Z"
```

### 5. Accès à l'application via un port-forward

Depuis un premier terminal lancez la commande suivante:

```
$ kubectl port-forward whoami 8888:80
```

Depuis un second terminal, vérifiez que l'application est accessible sur localhost depuis le port 8888:

```
$ curl localhost:8888
Hostname: whoami
IP: 127.0.0.1
IP: 10.244.1.4
RemoteAddr: 127.0.0.1:51562
GET / HTTP/1.1
Host: localhost:8888
User-Agent: curl/7.64.1
Accept: */*
```


### 6. Suppression du Pod

Le Pod peut etre supprimé avec la commande suivante:

```
$ kubectl delete po/whoami
```
