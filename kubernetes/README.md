## Tilt

Start and deploy the applicatoin with Tilt:
```bash
cd /Users/capa/Projects/PolarBookshop/catalog-service
tilt up
```

Undeploy the applicatoin with Tilt:
```bash
cd /Users/capa/Projects/PolarBookshop/catalog-service
tilt down
```

# Customize
Deploy the application via Kustomize:
```bash
kubectl apply -k .
```

Get pods for catalog-services
```bash
kubectl get pod -l app=catalog-service
```

Check the application logs for catalog-service:
```bash
kubectl logs deployment/catalog-service
```

Check which node each Pod has been allocated:
```bash
kubectl get pod -o wide
```

To visualize the sequence of events use this command on a separate Terminal window before applying the Kustomization:
```bash
kubectl get pods -l app=catalog-service --watch
```

List of all the contexts available:
```
kubectl config get-contexts
```

Change the context:
```
kubectl config use-context <context-name>
```

# Digital Ocean
Use the API token to grant doctl access to your DigitalOcean account. 
Pass in the token string when prompted by doctl auth init, and give this authentication context a name.
```
doctl auth init --context <NAME>

doctl auth init
```

Authentication contexts let you switch between multiple authenticated accounts. 
You can repeat steps 2 and 3 to add other DigitalOcean accounts, then list and switch between authentication contexts:
```
doctl auth list
doctl auth switch --context <NAME>
```


initialize a Kubernetes cluster using DigitalOcean Kubernetes (DOKS). It will be composed of three worker nodes, 
for which you can decide the technical specifications. You can choose between different options in terms of CPU, 
memory, and architecture. I’ll use nodes with 2 vCPU and 4 GB of memory:
```
doctl k8s cluster create polar-cluster \
--node-pool "name=basicnp;size=s-2vcpu-4gb;count=3;label=type=basic;" \
--region sfo3
```

Fetch the cluster ID:
```bash
doctl k8s cluster list
```

Current context:
```
kubectl config current-context
```

Create a new PostgreSQL server named polar-postgres:
```
doctl databases create polar-db \
    --engine pg \
    --region sfo3 \
    --version 14
```
**Custer ID: 574446cc-d22f-46e5-b63b-63dcf1bd5461**

Databases list:
```
doctl databases list
```
**Database ID: d18651d8-3bba-4695-b0b3-33c13d69241f**

Configure the firewall and secure access to the database server:
```
doctl databases firewalls append d18651d8-3bba-4695-b0b3-33c13d69241f --rule k8s:574446cc-d22f-46e5-b63b-63dcf1bd5461
```

Create two databases to be used by Catalog Service (polardb_catalog) and Order Service (polardb_order):
```
doctl databases db create d18651d8-3bba-4695-b0b3-33c13d69241f polardb_catalog

doctl databases db create d18651d8-3bba-4695-b0b3-33c13d69241f polardb_order
```

Retrieve the details for connecting to PostgreSQL:
```
doctl databases connection d18651d8-3bba-4695-b0b3-33c13d69241f --format Host,Port,User,Password

```

Create some Secrets in the Kubernetes cluster with the PostgreSQL credentials:
For Catalog Service:
```
kubectl create secret generic polar-postgres-catalog-credentials \
    --from-literal=spring.datasource.url=jdbc:postgresql://polar-db-do-user-25874083-0.f.db.ondigitalocean.com:25060/polardb_catalog \
    --from-literal=spring.datasource.username= \
    --from-literal=spring.datasource.password=
```

For Order Service:
```
kubectl create secret generic polar-postgres-order-credentials \
    --from-literal="spring.flyway.url=jdbc:postgresql://polar-db-do-user-25874083-0.f.db.ondigitalocean.com:25060/polardb_order" \
    --from-literal="spring.r2dbc.url=r2dbc:postgresql://polar-db-do-user-25874083-0.f.db.ondigitalocean.com:25060/polardb_order?ssl=true&sslMode=require" \
    --from-literal=spring.r2dbc.username= \
    --from-literal=spring.r2dbc.password=
```

Create a new Redis server named polar-redis:
**Note: It does not work, I had to create a Valley DB in the web console**
```
doctl databases create polar-redis --engine redis --region sfo3 --version 7
```

Configure a firewall so that the Redis server is only accessible from the Kubernetes cluster:
```
doctl databases firewalls append d12bc696-f971-4d77-aefa-c73ff79f4f43 --rule k8s:574446cc-d22f-46e5-b63b-63dcf1bd5461
```

Retrieve the details for connecting to Redis:
```
doctl databases connection d12bc696-f971-4d77-aefa-c73ff79f4f43 --format Host,Port,User,Password
```

Secret in the Kubernetes cluster with the Redis credentials:
```
kubectl create secret generic polar-redis-credentials \
    --from-literal=spring.redis.host=polar-redis-do-user-25874083-0.f.db.ondigitalocean.com \
    --from-literal=spring.redis.port=25061 \
    --from-literal=spring.redis.username= \
    --from-literal=spring.redis.password= \
    --from-literal=spring.redis.ssl=true
```



kustomize edit set image catalog-service=ghcr.io/cesar-alejandro-perez/catalog-service:b712eb76bfde7413dd919d94e139e7c2ec110965