# Apache Spark on Kubernetes: Step-by-Step Setup and Troubleshooting
**This guide provides a comprehensive step-by-step approach to setting up Apache Spark on a Kubernetes cluster. It includes both the installation and common troubleshooting steps to ensure a smooth deployment.**

### Prerequisites

1. Kubernetes cluster is already set up. (Refer to our previous [guide](https://github.com/dhanush-nferx/vgrt-fusion-k8s) for Kubernetes setup if needed)
2. Helm is installed and configured on your local machine.
3. Docker is installed and running.

## Step 1: Add Helm Repository

First, add the Bitnami Helm repository to your Helm configuration.

```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

## Step 2: Create Namespace for Spark

Create a dedicated namespace for the Spark deployment.

```
kubectl create namespace spark
```

## Step 3: Create Values File for Custom Configuration

Create a values.yaml file to customize your Spark installation. Here’s an example configuration:

```
master:
  replicaCount: 1
  nodeSelector:
    spark-role: master

worker:
  replicaCount: 2
  nodeSelector:
    spark-role: worker

image:
  registry: docker.io
  repository: bitnami/spark
  tag: 3.5.1-debian-12-r8

imagePullSecrets:
  - name: myregistrykey
```

> [!NOTE]
> imagePullSecrets is necessary if you are using a private Docker registry or need to authenticate with Docker Hub.

## Step 4: Install Apache Spark with Helm

Use the Helm chart to install Apache Spark on your Kubernetes cluster.

```
helm install spark bitnami/spark --namespace spark -f values.yaml
```

## Step 5: Verify the Deployment

Check if the Spark master and workers are running correctly.

```
kubectl get pods -n spark
```

You should see output similar to:

<img width="523" alt="image" src="https://github.com/user-attachments/assets/e739f1e5-0027-45dd-9021-7f694ab6d63a">

## Step 6: Access the Spark UI

**Port-Forward the Spark Master UI**

Since the Spark master UI is exposed on port 80 by default, use port-forwarding to access it locally.

```
kubectl port-forward svc/spark-master-svc -n spark 8080:80
```

Open your browser and navigate to http://localhost:8080 to access the Spark UI.

# Troubleshooting Common Issues and Solutions

**Issue: Pod Scheduling Failures**

If pods are not scheduling, you might see errors related to taints and node affinity.

1. Remove Unnecessary Taints:
   ```
   kubectl taint nodes controlplane node-role.kubernetes.io/control-plane-
   ```
2. Verify Node Labels:
   
   Ensure node labels match the nodeSelector in your Helm values file.

   You can label your nodes with:
   
   ```
   kubectl label nodes controlplane kubernetes.io/hostname=controlplane
   kubectl label nodes node01 kubernetes.io/hostname=node01
   kubectl label nodes node02 kubernetes.io/hostname=node02
   ```
   
**Issue: Spark UI Not Accessible**

If the Spark UI is not accessible, ensure the correct port is used:

1. Verify Service Ports:
   
   Check the Spark master service ports:

   ```
   kubectl get services -n spark
   ```
   
2. Port-Forward Correctly:
   
   Use the correct service port for port-forwarding:

   ```
   kubectl port-forward svc/spark-master-svc -n spark 8080:80
   ```

***By following this guide, you should have a fully functional Apache Spark setup on your Kubernetes cluster.***

> [!IMPORTANT]
> ***If you have any queries, post a comment and we will look into it.***
