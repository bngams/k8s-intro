# Créer un ConfigMap manuellement

Créer un fichier xxx.yaml

```
kind: ConfigMap 
apiVersion: v1 
metadata:
  name: example-configmap 
data:
  # Configuration values can be set as key-value properties
  database: mongodb
  database_uri: mongodb://localhost:27017
  
  # Or set as complete file contents (even JSON!)
  keys: | 
    image.public.key=771 
    rsa.public.key=42
```

puis exécuter la commande:

```kubectl apply -f <my-config-map.yaml> ```


# Créer un ConfigMap et le monter depuis un volume

....
