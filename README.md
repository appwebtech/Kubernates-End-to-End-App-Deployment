<p float="left">
  <img src="https://github.com/appwebtech/EKS-Cluster-With-Terraform/blob/main/images/k8s-logo.png" width="80">
  <img src="https://github.com/appwebtech/Deploy-Docker-With-Terraform/blob/main/images/docker.png" width="100">
</p>

----

# End-to-End Application Deployment with K8s

## Introduction

In this project, I will be deploying end-to-end mongo-express and mongoDB applications using K8s. This project mimics a production environment where a web tier (mongo-express) uses CRUD to manage objects in a database tier (MongoDB). Mongo Express is a lightweight web-based administrative UI used to manage MongoDB databases interactively. It's designed using Express JS, Twitter Bootstrap and Node.js technologies.

## Tech Stack Architecture

I will need two pods to house the two applications (MongoDB pod and Mongo Express pod). MongoDB pod will use an **Internal Service** whilst Mongo Express will need a **Deployment.yaml** descriptor and will need to be externally exposed via an **External Service** for use via HTTP requests in a browser. I'll create a **ConfigMap** which will house the *DB url* and a **Secret** which will house the *DB User* and *DB Password* which will be referenced in the descriptor file.

### Browser Request Flow

1. Client inputs data in **Mongo Express UI**
2. **Mongo Express External Service** relays request to **Mongo Express Pod**
3. **Mongo Express Pod** relays requests to **MongoDB Internal Service**
4. **MongoDB Internal Service** relays request to **MongoDB Pod**
5. **MongoDB Pod** authenticates request via **Secret**

### MongoDB Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb-deployment
  labels:
    app: mongodb
spec:
  replicas: 1
  selector:
    matchLabels:
        app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom:
            secretKeyRef:
                name: mongodb-secret
                key: mongo-root-username
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
              secretKeyRef:
                name: mongodb-secret
                key: mongo-root-password
```

