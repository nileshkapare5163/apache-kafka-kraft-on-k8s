# Kafka KRaft Deployment on Kubernetes (kubeadm) -- Azure Ubuntu 24.04

This repository provides a complete step-by-step workflow for deploying
**Apache Kafka in KRaft mode (without Zookeeper)** on a **Kubernetes
cluster created using kubeadm** on **Ubuntu 24.04**.\
The entire infrastructure runs on **Microsoft Azure Virtual Machines**,
with Kafka exposed externally for real-world Producer/Consumer
applications.

## Overview

This project demonstrates:

-   Setting up a Kubernetes cluster (1 Master + 1 Worker) using kubeadm\
-   Installing required components (Docker, Java, Kubernetes tools)\
-   Deploying Kafka in **KRaft mode** using Kubernetes manifests\
-   Enabling external access through NodePort\
-   Running external Producer/Consumer apps (Node.js / Python)\
-   Using Azure VMs as the underlying compute layer\
-   Networking handled via **Calico CNI**

## Architecture Summary

  Component                 Details
  ------------------------- -----------------------------
  **Cloud Platform**        Azure VM (Ubuntu 24.04 LTS)
  **Cluster Type**          Kubernetes via kubeadm
  **Nodes**                 1 Master • 1 Worker
  **Kafka Mode**            KRaft (No Zookeeper)
  **Access Method**         NodePort (30092 / 30093)
  **Network Plugin**        Calico
  **Producers/Consumers**   External (Node.js/Python)

## Azure VM Configuration

### Master Node

-   VM Size: **D4s v3**
-   **4 vCPU**, **16 GB RAM**
-   Disk: **30 GB SSD**
-   Inbound rules:
    -   22 (SSH)
    -   6443 (Kubernetes API)
    -   30092, 30093 (Kafka NodePort)

### Worker Node

-   VM Size: **D2s v3**
-   **2 vCPU**, **8 GB RAM**
-   Disk: **20 GB**
-   Inbound rules:
    -   22 (SSH)

## Kubernetes Cluster Setup

``` bash
sudo apt update
sudo apt install docker.io -y
sudo apt install openjdk-17-jdk -y
sudo apt install kubeadm kubelet kubectl -y
```

Initialize Kubernetes on master:

``` bash
sudo kubeadm init
```

Configure kubectl:

``` bash
mkdir -p $HOME/.kube
cp /etc/kubernetes/admin.conf $HOME/.kube/config
```

Install Calico CNI:

``` bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

Join worker node:

``` bash
kubeadm join <MASTER_IP>:6443 --token <TOKEN> --discovery-token-ca-cert-hash <HASH>
```

Allow scheduling pods on master:

``` bash
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

## Deploying Kafka in KRaft Mode

``` bash
mkdir -p kafka-kraft && cd kafka-kraft
```

Add Kubernetes manifests (ConfigMap, StatefulSet, Service, PVC,
StorageClass).

Apply:

``` bash
kubectl apply -f .
```

Check:

``` bash
kubectl get pods -n kafka
kubectl get svc -n kafka
```

## Kafka External Access

  Port        Purpose
  ----------- ---------------------------------
  **30092**   External listener for producers
  **30093**   Internal broker communication

Example:

    PLAINTEXT://<PUBLIC_IP>:30092

## Testing with Producers & Consumers

### Node.js Example

``` javascript
const { Kafka } = require("kafkajs");

const kafka = new Kafka({
  brokers: ["<PUBLIC_IP>:30092"]
});
```

### Python Example

``` python
from kafka import KafkaProducer
producer = KafkaProducer(bootstrap_servers='<PUBLIC_IP>:30092')
```

## Project Structure

    kafka-kraft/
     ├── kafka-configmap.yaml
     ├── kafka-statefulset.yaml
     ├── kafka-service.yaml
     ├── storage-class.yaml
     └── persistent-volume-claim.yaml

## Conclusion

This project provides a practical, cloud-ready, and modern Kafka
deployment on Kubernetes using the KRaft architecture. It is suitable
for learning, DevOps practices, POCs, or preparing for real
production-grade Kafka setups.
