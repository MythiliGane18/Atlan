
# E-Commerce Application Deployment on Kubernetes

This README provides instructions on deploying and managing the simulated e-commerce application in a Kubernetes cluster. The application consists of a frontend (React served by Nginx), a backend (Node.js microservices), MongoDB for data persistence, and RabbitMQ for asynchronous order processing. This guide includes architecture decisions, deployment instructions, and assumptions made.

## Table of Contents

- [Architecture Overview](#architecture-overview)
- [Prerequisites](#prerequisites)
- [Architecture and Design Decisions](#architecture-and-design-decisions)
- [Deployment Steps](#deployment-steps)
  - [Step 1: Setting up ConfigMaps and Secrets](#step-1-setting-up-configmaps-and-secrets)
  - [Step 2: Deploy MongoDB](#step-2-deploy-mongodb)
  - [Step 3: Deploy RabbitMQ](#step-3-deploy-rabbitmq)
  - [Step 4: Deploy Backend API](#step-4-deploy-backend-api)
  - [Step 5: Deploy Frontend](#step-5-deploy-frontend)
  - [Step 6: Configure Ingress for External Access](#step-6-configure-ingress-for-external-access)
- [Networking and Service Discovery](#networking-and-service-discovery)
- [Persistence Strategy](#persistence-strategy)
- [Security Considerations](#security-considerations)
- [Scaling and Autoscaling](#scaling-and-autoscaling)
- [Monitoring and Logging](#monitoring-and-logging)
- [Troubleshooting Guide](#troubleshooting-guide)

## Architecture Overview

The e-commerce application consists of the following components:
1. **Frontend**: React-based web app served by Nginx.
2. **Backend**: Node.js microservices exposing REST APIs.
3. **MongoDB**: NoSQL database for persistence.
4. **RabbitMQ**: Message broker for async order processing.

The Kubernetes deployment ensures high availability, scalability, and security for each component using a microservices architecture.

## Prerequisites

Before deploying the application, ensure that the following prerequisites are met:
1. **Kubernetes Cluster**: A running Kubernetes cluster (minikube, GKE, EKS, etc.).
2. **kubectl**: Installed and configured to interact with your cluster.
3. **Ingress Controller**: An Nginx Ingress controller should be deployed in the cluster for external access to services.
4. **Persistent Storage**: Storage provisioner for handling Persistent Volumes.

## Architecture and Design Decisions

### **1. Component Breakdown**

- **Frontend**: React-based web app served via an Nginx web server. The frontend communicates with the backend API for business operations.
- **Backend**: Node.js microservices that provide REST APIs for frontend services, handle business logic, and interact with MongoDB and RabbitMQ.
- **MongoDB**: A NoSQL database deployed as a StatefulSet with Persistent Volume Claims (PVCs) for data persistence.
- **RabbitMQ**: A message queue service responsible for asynchronous order processing. It also uses Persistent Volumes for durability.

### **2. Kubernetes Resources**

- **Deployments**:
    - Used for stateless applications (Frontend, Backend).
    - Ensures rolling updates for zero downtime.

- **StatefulSets**:
    - Used for stateful applications (MongoDB, RabbitMQ).
    - Provides stable network identifiers and persistent storage.

- **ConfigMaps and Secrets**:
    - **ConfigMaps** are used for non-sensitive configuration data (e.g., Nginx config, environment variables).
    - **Secrets** are used for sensitive information (e.g., MongoDB connection strings).

- **Persistent Volumes (PVs) and Persistent Volume Claims (PVCs)**:
    - MongoDB and RabbitMQ require data to persist beyond the lifecycle of individual Pods. PVCs are bound to dynamically provisioned PVs to ensure this.

### **3. Ingress and Service Networking**

- **Ingress**: The Ingress controller provides external access to the frontend and backend services. It uses domain-based routing for distinguishing between frontend and backend requests.
- **Services**: ClusterIP services are used for internal communication between components (e.g., Backend communicating with MongoDB).

### **4. Scaling and High Availability**

- **Horizontal Pod Autoscaling (HPA)**: Autoscaling is implemented for the frontend and backend services based on CPU utilization. This ensures that the application can scale out when the demand increases.
- **ReplicaSets**: Each stateless application (frontend, backend) runs with a minimum of 3 replicas to ensure high availability.
- **StatefulSet for MongoDB**: Ensures data replication and fault tolerance by running MongoDB with multiple replicas.


## Deployment Steps

### Step 1: Setting up ConfigMaps and Secrets

Before deploying application, set up ConfigMaps and Secrets to handle application configurations and sensitive data.

- **Frontend ConfigMap**: Contains environment variables such as the backend API URL (`frontend/frontend-configmap.yaml`).
- **Backend ConfigMap & Secret**: ConfigMap holds non-sensitive environment variables, while Secrets store sensitive information like the database credentials (`backend/backend-configmap.yaml` & `backend/backend-secret.yaml`).
- **MongoDB ConfigMap & Secret**: Used to configure MongoDB initialization settings and credentials (`mongodb/mongodb-configmap.yaml` & `mongodb/mongodb-secret.yaml`).
- **RabbitMQ ConfigMap & Secret**: Stores RabbitMQ credentials and configuration (`rabbitmq/rabbitmq-configmap.yaml` & `rabbitmq/rabbitmq-secret.yaml`).

```bash
kubectl apply -f frontend/frontend-configmap.yaml
kubectl apply -f backend/backend-configmap.yaml
kubectl apply -f backend/backend-secret.yaml
kubectl apply -f mongodb/mongodb-configmap.yaml
kubectl apply -f mongodb/mongodb-secret.yaml
kubectl apply -f rabbitmq/rabbitmq-configmap.yaml
kubectl apply -f rabbitmq/rabbitmq-secret.yaml
```

### Step 2: Deploy MongoDB

Deploy the MongoDB StatefulSet with Persistent Volume Claims (PVCs) for data persistence. A headless service (`mongo-service`) is used for service discovery within the cluster.

```bash
kubectl apply -f mongodb/mongodb-statefulset.yaml
kubectl apply -f mongodb/mongodb-service.yaml
```

### Step 3: Deploy RabbitMQ

Deploy RabbitMQ with a StatefulSet and PVC for persistent storage, along with the necessary services for RabbitMQ communication.

```bash
kubectl apply -f rabbitmq/rabbitmq-statefulset.yaml
kubectl apply -f rabbitmq/rabbitmq-service.yaml
```

### Step 4: Deploy Backend API

Deploy the Node.js backend services, which communicate with MongoDB and RabbitMQ. This deployment is designed to autoscale based on traffic.

```bash
kubectl apply -f backend/backend-deployment.yaml
kubectl apply -f backend/backend-service.yaml
```

### Step 5: Deploy Frontend

Deploy the frontend (Nginx) to serve the React app. The frontend accesses the backend through the Kubernetes service.

```bash
kubectl apply -f frontend/frontend-deployment.yaml
kubectl apply -f frontend/frontend-service.yaml
```

### Step 6: Configure Ingress for External Access

Configure an Nginx Ingress to expose the frontend and backend services to external users.

```bash
kubectl apply -f ingress/ingress.yaml
```



## Networking and Service Discovery

- **Frontend and Backend Communication**: The frontend communicates with the backend through internal service discovery using the `backend-service` ClusterIP service.
- **Database Connection**: The backend connects to MongoDB via `mongo-service`.
- **Ingress**: External traffic is routed to the frontend and backend via the Nginx Ingress controller.

## Persistence Strategy

- **MongoDB and RabbitMQ**: Persistent Volumes are used to ensure data persistence and durability. MongoDB uses a StatefulSet for maintaining ordered identities, ensuring stable network IDs and storage.

## Security Considerations

- **Secrets Management**: Sensitive information like MongoDB credentials and RabbitMQ credentials are stored securely in Kubernetes Secrets.
- **Network Policies**: Network policies can be implemented to isolate internal services from external traffic, ensuring secure inter-service communication.

## Scaling and Autoscaling

- **Horizontal Pod Autoscaler (HPA)**: Autoscaling can be set up for the frontend and backend services based on CPU and memory utilization. You can modify the deployment YAMLs to add HPA configurations.

## Monitoring and Logging

- **Monitoring**: Integrate Prometheus and Grafana to monitor resource usage (CPU, memory, etc.) and application performance.
Deploy these files to your Kubernetes cluster:

```bash
kubectl apply -f monitoring/prometheus-configmap.yaml
kubectl apply -f monitoring/prometheus-deployment.yaml
kubectl apply -f monitoring/node-exporter.yaml
kubectl apply -f monitoring/grafana-deployment.yaml
```

Access the services via:

> Prometheus: 
> http://node-ip:30000
> 
> Grafana: 
>http://node-ip:32000

- **Logging**: The ELK stack (Elasticsearch, Logstash, Kibana) can be set up to aggregate logs across all services, making troubleshooting easier.
Deploy the ELK stack 

``` bash

kubectl apply -f logging/elasticsearch-deployment.yaml
kubectl apply -f logging/logstash-deployment.yaml
kubectl apply -f logging/logstash-config.yaml
kubectl apply -f logging/kibana-deployment.yaml
kubectl apply -f logging/filebeat-configmap.yaml
kubectl apply -f logging/filebeat-daemonset.yaml
```
Access the services via
> Kibana: 
> http://node-ip:5601
