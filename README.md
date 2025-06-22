# kubernetes-spark
Running Spark Locally using Kubernets

# Environment Installation Guide

This document provides instructions for setting up the lab environment used by SPOK (Smart Platform for Orchestrated Knowledge). The installation use Helm charts.

## Installation with Helm Charts

Some components are installed using Helm charts. These include both official charts and charts derived from open-source documentation.

### Step 1: Add Helm repositorys

```sh
# Install the repository
helm repo add spark-operator https://kubeflow.github.io/spark-operator
helm repo add minio-operator https://operator.min.io
helm repo add prometheus https://prometheus-community.github.io/helm-charts

# Update the repository
helm repo update
```

### Step 2: Download Charts

```sh
# Download and extract Helm charts to source directory
helm pull spark-operator/spark-operator --version 2.1.1 --untar --untardir ./helm-charts
helm pull minio-operator/operator --version 7.0.0 --untar --untardir ./helm-charts
helm pull minio-operator/tenant --version 7.0.0 --untar --untardir ./helm-charts
helm pull prometheus/kube-prometheus-stack --version 69.3.2 --untar --untardir ./helm-charts
```

### Step 3: Create Kubernetes Environment

```sh
# Start the minikube
minikube start --cpus=4 --memory=8G

# Enable the metrics server
minikube addons enable metrics-server


# Create the namespace to be used
kubectl create namespace processing
kubectl create namespace deepstore
kubectl create namespace monitoring
```

### Step 4: Install Applications Using Helm

Replace paths and values according to the application you are installing.

```sh
# Running Spark Operator
helm install spark-operator ./helm-charts/spark-operator -f ./helm-charts/spark-operator/values.yaml -n processing

# Running MinIO Operator
helm install minio-operator ./helm-charts/operator -f ./helm-charts/operator/values.yaml -n deepstore

# Running MinIO Tenant
helm install minio-tenant ./helm-charts/tenant -f ./helm-charts/tenant/values.yaml -n deepstore

# Running Prometheus Operator
helm install prometheus-operator ./helm-charts/kube-prometheus-stack -f ./helm-charts/kube-prometheus-stack/values.yaml -n monitoring

```

### Step 5: Build the Docker image with the spark app

```sh
# Navigate to the images folder src/images/spark and build the image
docker build -t igorformiga/spark-kubernets:1.0 .

# Push the image to the registry
docker push igorformiga/spark-kubernets:1.0 
```


### Step 6: Generate the datasets

```sh
# Apply the shadowtraffic manifest
kubectl delete -f src/datagen/shadowtraffic-ubereats.yaml -n default
```


### Step 7: Run the example

```sh
kubectl apply -f src/test_exemples/secret-credential.yaml -n processing

kubectl apply -f src/test_exemples/service-metrics.yaml -n processing

# Check if the Git SSH keys are correct. Change them to match your repository if you encounter permission issues.
# Example:
#         kubectl create secret generic git-creds --from-file=ssh=~/.ssh/id_rsa --from-file=known_hosts=~/.ssh/known_hosts --dry-run=client -o yaml
# kubectl apply -f labs/lab-10/scripts/secret-git-credential.yaml -n processing

kubectl apply -f src/test_exemples/spark-application.yaml -n processing

```

### Configs:

- minio: 9090:9090 (http://localhost:9090/)
- prometheus: 9099:9090 (http://localhost:9099/)
- grafana: 3000:3000 (http://localhost:3000/)