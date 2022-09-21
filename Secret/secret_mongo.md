# Création et utilisation d'un Secret

## Exercice

Dans cet exercice vous allez utiliser un Secret pour vous connecter à une base de données externe.

### 1. Le context

L'image *lucj/messages:1.0* contient une application simple qui écoute sur le port 80 et permet, via des requêtes HTTP, de créer des messages ou de lister les messages existants.

Ces messages sont sauvegardés dans une base de données *MongoDB* dont l'URL de connexion doit être fournie à l'application de façon à ce que celle-ci puisse s'y connecter. Nous pouvons lui fournir via une variable d'environnement MONGODB_URL ou via un fichier texte accessible depuis */run/secrets/MONGODB_URL*.

### 2. La base de données

Pour cet exercice la base de données suivante sera utilisée:

- host: *db.techwhale.io*
- port: *27017* (port par défaut de Mongo)
- nom de la base: *message*
- tls activé
- utilisateur: *k8sExercice* / *k8sExercice*

L'URL de connection est donc la suivante:

```
mongodb://k8sExercice:k8sExercice@db.techwhale.io:27017/message?ssl=true&authSource=admin
```

### 3. Création du Secret

Créez un Secret nommé *mongo*, le champ *data* de celui-ci doit contenir la clé *mongo_url* dont la valeur est la chaine de connection spécifiée ci-dessus.

Choisissez pour cela l'une des options suivantes:

- Option 1: utilisation de la commande `kubectl create secret generic` avec l'option `--from-file`

- Option 2: utilisation de la commande `kubectl create secret generic` avec l'option `--from-literal`

- Option 3: utilisation d'un fichier de spécification

### 4. Utilisation du Secret dans une variable d'environnement

Définissez un Pod nommé *api-env* dont l'unique container a la spécification suivante:

- nom: *api*
- image: *lucj/messages:1.0*
- une variable d'environnement *MONGODB_URL* ayant la valeur liée à la clé *mongo_url* du Secret *mongo* créé précédemment

Créez ensuite ce Pod et exposez le en utilisant la commande `kubectl port-forward` en faisant en sorte que le port 8888 de votre machine locale soit mappé sur le port 80 du Pod *api-env*.

Depuis un autre terminal, vérifiez que vous pouvez créer un message avec la commande suivante:

Note: assurez vous de remplacer *YOUR_NAME* par votre prénom

```
$ curl -H 'Content-Type: application/json' -XPOST -d '{"from":"YOUR_NAME", "msg":"hello"}' http://localhost:8888/messages
```

### 5. Utilisation du Secret dans un volume

Définissez un Pod nommé *api-vol* ayant la spécification suivante:

- un volume nommé *mongo-creds* basé sur le Secret *mongo* dont la clé *mongo_url* est renommée *MONGODB_URL* (utilisation du couple (key,path) sous la clé secret/items)
- un container ayant la spécification suivante:
  - nom: *api*
  - image: *lucj/messages:1.0*
  - une instructions *volumeMounts* permettant de monter le volume *mongo-creds* sur le path */run/secrets*

Créez le Pod et vérifier que vous pouvez créer un message de la même façon que dans le point précédent en exposant le Pod via un *port-forward*.

---

## Correction

### 3. Création du Secret

- Option 1: utilisation de la commande `kubectl create secret generic` avec l'option `--from-file`

Utilisez la commande suivante afin de créer un fichier *mongo_url* contenant la chaine de connexion à la base de données:

```
echo -n "mongodb://k8sExercice:k8sExercice@db.techwhale.io:27017/message?ssl=true&authSource=admin" > mongo_url
```

Nous crééons ensuite le Secret à partir de ce fichier:

```
kubectl create secret generic mongo --from-file=mongo_url
```

- Option 2: utilisation de la commande `kubectl create secret generic` avec l'option `--from-literal`

La commande suivante permet de créer le Secret à partir de valeurs littérales

```
kubectl create secret generic mongo \
--from-literal=mongo_url='mongodb://k8sExercice:k8sExercice@db.techwhale.io:27017/message?ssl=true&authSource=admin'
```

- Option 3: utilisation d'un fichier de spécification

La première étape est d'encrypter en base64 la chaine de connexion

```
$ echo -n 'mongodb://k8sExercice:k8sExercice@db.techwhale.io:27017/message?ssl=true&authSource=admin' | base64

bW9uZ29kYjovL2s4c...yY2U9YWRtaW4=
```

Ensuite nous pouvons définir le fichier de spécification mongo-secret.yaml:

```
apiVersion: v1
kind: Secret
metadata:
  name: mongo
data:
  mongo_url: bW9uZ29kYjovL2s4c...yY2U9YWRtaW4=
```

La dernière étape consiste à créer le Secret à partir de ce fichier

```
$ kubectl create -f mongo-secret.yaml
secret "mongo" created
```

### 4. Utilisation du Secret dans une variable d'environnement

Nous définissons la spécification suivante dans le fichier *pod_messages_env.yaml*

```
apiVersion: v1
kind: Pod
metadata:
  name: api-env
spec:
  containers:
  - name: api
    image: lucj/messages:1.0
    env:
    - name: MONGODB_URL
      valueFrom:
        secretKeyRef:
          name: mongo
          key: mongo_url
```

Nous pouvons alors créer le Pod:

```
$ kubectl create -f pod_messages_env.yaml
pod "api-env" created
```

La commande suivante permet d'exposer en localhost l'API tournant dans le container du Pod:

```
$ kubectl port-forward api-env 8888:80
Forwarding from 127.0.0.1:8888 -> 80
...
```

Depuis un autre terminal sur notre machine locale, nous pouvons alors envoyer une requête POST sur l'API:

Note: assurez vous de remplacer *YOUR_NAME* par votre prénom

```
$ curl -H 'Content-Type: application/json' -XPOST -d '{"from":"YOUR_NAME", "msg":"hello"}' http://localhost:8888/messages
```

Vous recevrez une réponse en json similaire à celle ci-dessous:
```
{"from":"YOUR_NAME","msg":"hello","at":"2020-09-03T12:45:07.688Z","_id":"5ac37753dfe0ee000f9b65e0"}
```

### 5. Utilisation du Secret dans un volume

Nous définissons la spécification suivante dans le fichier *pod_messages_vol.yaml*

```
apiVersion: v1
kind: Pod
metadata:
  name: api-vol
spec:
  containers:
  - name: api
    image: lucj/messages:1.0
    volumeMounts:
    - name: mongo-creds
      mountPath: "/run/secrets"
      readOnly: true
  volumes:
  - name: mongo-creds
    secret:
      secretName: mongo
      items:
      - key: mongo_url
        path: MONGODB_URL
```

Nous pouvons alors créer le Pod:

```
$ kubectl create -f pod_messages_vol.yaml
pod "api-vol" created
```

La commande suivante permet d'exposer en localhost l'API tournant dans le container du Pod:

```
$ kubectl port-forward api-vol 8889:80
Forwarding from 127.0.0.1:8889 -> 80
...
```

Depuis la machine locale, nous pouvons alors envoyer une requête POST sur l'API:

```
$ curl -H 'Content-Type: application/json' -XPOST -d '{"from":"me", "msg":"hello"}' http://localhost:8889/messages
```

Si tout s'est bien passé, nous obtenons une réponse ayant le format suivant:

```
{"from":"me","msg":"hola","at":"2020-09-03T13:05:11.408Z","_id":"5ac37c07bace38000f9b09e2"}
```
