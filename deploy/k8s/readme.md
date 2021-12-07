# Install K8 Cluster

- [Install K8 Cluster](#install-k8-cluster)
  - [Install Prometheus Monitoring](#install-prometheus-monitoring)
    - [Prometheus Dashboard](#prometheus-dashboard)
    - [AlertManager Dashboard](#alertmanager-dashboard)
    - [Grafana Dashboard](#grafana-dashboard)
  - [Install Emissary Ingress](#install-emissary-ingress)
  - [MongoDB Server](#mongodb-server)
    - [To install the MongoDB service](#to-install-the-mongodb-service)
    - [Connect to the Mongo service](#connect-to-the-mongo-service)
  - [Kafka Service Broker](#kafka-service-broker)
    - [Install Kafka Service Broker](#install-kafka-service-broker)
    - [Connect to the Kafka Service Broker](#connect-to-the-kafka-service-broker)
---
## Install Prometheus Monitoring

First install the prometheus operator

```bash
kubectl apply -f ./monitoring/setup
```

If the operator is installed apply the resources

```bash
kubectl apply -f ./monitoring
```

This will install Prometheus AlertManager and Grafana

To Access the dashboards use the following commands

### Prometheus Dashboard
```bash
kubectl port-forward svc/prometheus-k8s 9090 -n monitoring 
```

[Prometheus Dashboard](http://localhost:9090)

### AlertManager Dashboard

```bash
kubectl port-forward svc/alertmanager-main 9093 -n monitoring 
```

[AlertManager Dashboard](http://localhost:9093)

### Grafana Dashboard

```bash
kubectl port-forward svc/grafana 3000 -n monitoring 
```

[Grafana Dashboard](http://localhost:3000)
*use username: 'admin' and password: 'admin'*

---

## Install Emissary Ingress

To install the emissary ingress run the following.

```bash
kubectl apply -f ./emissary
``` 

---
## MongoDB Server

### To install the MongoDB service

Before you install please change the default username and password in the ``03-secrets.yaml`` file.

the username is default set to: ``adminuser``

the password is default set to ``password123``

```bash
kubectl apply -f ./mongodb
```

### Connect to the Mongo service
To connect to the Mongo service use run the following command to allow access from local machine

```bash
kubectl port-forward service/mongo-nodeport-svc 27017:27017 -n mongo
```

Use the following connection string to connect to the mongodb service

``mongodb://adminuser:password123@localhost:27017``

---
## Kafka Service Broker

### Install Kafka Service Broker

To install the Kafka Service Broker run the following

```bash
kubectl apply -f ./kafka
```

**You probably need to run it twice the first time because of the ``Custom Resource Definitions`` probably are not ready yet.**

### Connect to the Kafka Service Broker

To Connect to the kafka service broker for local development use the following command

```bash
kubectl port-forward services/kafka-cluster-kafka-external-bootstrap 9092:9094 -n kafka
```

This will open port 9092 to connect to the kafka service broker

use ``localhost:9092`` as bootstrap server

---

