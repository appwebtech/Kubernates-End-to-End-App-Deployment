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

Before creating the deployment, Ill create the secret but instead of hard-coding the values of the username and password, I have encrypted them in base64 using my terminal and added the hashes in my secret file.

```yaml 
apiVersion: v1
kind: Secret
metadata:
  name: mongodb-secret
type: Opaque
data:
    mongo-root-username: am9zZW1iaQ==
    mongo-root-password: Y2F0aGVkcmFs
```

In bash, I'll apply the secret descriptor file and subsequently the mongo descriptor as well. See below the container is creating. 

![image-1](./images/image-1)

We can actually describe the pod and see what is happening under the hood. Mongo ismage is still getting pulled from dockerhub...

![image-2](./images/image-2)

### MongoDB Internal Service

It's time to create the internal service for MongoDB to enable requests to and from Mongo by other services. I'll amalgamate the **Service** descriptor file with the mongo deployment because mongo is a dependency of the service without which it won't relay requests to Mongo Express.

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: mongodb-service
spec:
  selector:
    app: mongodb
  ports:
    - protocol: TCP
      port: 27017
      targetPort: 27017
```

<details>
  <summary>Click to View Complete File</summary>
  
  ### Console Output
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
---
apiVersion: v1
kind: Service
metadata:
  name: mongodb-service
spec:
  selector:
    app: mongodb
  ports:
    - protocol: TCP
      port: 27017
      targetPort: 27017

```
</details>

Upon applying the descriptor yaml file, the *mongodb-deployment* will remain unchanged as I had run it previously. K8s is idempotent or smart enough to know I applied the configuration previously so it won't apply it again.

![image-3](./images/image-3)

![image-4](./images/image-4)

