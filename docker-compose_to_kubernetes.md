# Docker Compose to Kubernetes Migration

<mark>Requires Velocity 1.5.2 or later.</mark>

## Dump Current Database from Existing Installation

  1. Open a shell inside the database container
   
    docker exec -it velocity_database_1 bash

  2. Dump all data from the existing database
   
    mongodump

  3. Leave the container shell
   
    exit

  4. Copy the `dump` folder out of the container
   
    docker cp velocity_database_1:/dump <DESTINATION>

## Set Up MongoDB in Kubernetes

  1. Install the MongoDB Helm chart with a root password of your choice
   
    helm install --set mongodbRootPassword=<ROOT_PASSWORD> --name velocity-mongo stable/mongodb

  2. Open a shell inside the MongoDB pod

    kubectl exec -it --namespace default svc/velocity-mongo-mongodb /bin/bash

  3. Open a Mongo Shell session as the root user

    mongo admin -u root -p <ROOT_PASSWORD>

  4. Create a non-root user with sufficient privileges for use in Velocity

    db.createUser({user: "<NEW_USERNAME>", pwd: "<NEW_PASSWORD>", roles: [{role: "readWriteAnyDatabase", db: "admin"}, {role: "dbAdminAnyDatabase", db: "admin"}, {role: "clusterAdmin", db: "admin"}]})
  
  5. Exit the Mongo Shell

    exit

## Restore the Database Dump to the MongoDB Pod

  1. Identify name of MongoDB pod

    kubectl get pods

  2. Copy database dump folder into MongoDB pod

    kubectl cp <DUMP_LOCATION> <FULL_POD_NAME>:/tmp/dump

  3. Open a shell inside the MongoDB pod

    kubectl exec -it --namespace default svc/velocity-mongo-mongodb /bin/bash 

  4. Navigate to the dump directory

    cd /tmp/dump

  5. List the folders; each folder is a database within MongoDB

    ls

  6. Restore each database EXCEPT `ADMIN`

    mongorestore --uri="mongodb://<NEW_USERNAME>:<NEW_PASSWORD>@localhost:27017/?authSource=admin" <DB_NAME> -d <DB_NAME>

  7. Exit the pod shell

    exit

## Configure SSL Certificate

  1. Create a file called `velocity-secret.yml` with the following contents
   
  ```
  apiVersion: v1
  data:
    tls.crt: <BASE64_CERT>
    tls.key: <BASE64_KEY>
  kind: Secret
  metadata:
    name: velocitytls
    namespace: default
  type: Opaque
  ```

  2. Encode the certificate and key files and paste them into the appropriate spaces

    cat <FILE>.pem | base64

  3. Apply the `velocity-secret.yml` file

    kubectl apply -f velocity-secret.yml

## Configure Kubernetes Ingress

  1. Apply 2 Ingress-Nginx yml files

    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/mandatory.yaml
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/provider/cloud-generic.yaml

## Install Velocity
**Before proceeding, please ensure that the `settings.json` file generated the last time you ran the Velocity installer is present in the ~/.ucv directory**

  1. Run the Velocity installer. Fields should auto-populate from the `settings.json` file, but you can change the install location and hostname if you wish. Select Kubernetes as the platform. This will produce a Helm chart tgz you can use to install Velocity.
  2. Install the Velocity Helm chart.

    helm install --set license=accept --set url.domain=<HOSTNAME> --set mongo.url=mongodb://<NEW_USERNAME>:<NEW_PASSWORD>@velocity-mongo-mongodb:27017/?authSource=admin --name velocity <VELOCITY_HELM_CHART>

**Velocity should now be running**
