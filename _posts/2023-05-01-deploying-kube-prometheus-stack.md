---
title: Deploying kube-prometheus-stack with persistent storage on Kubernetes Cluster
date: 2023-05-01 18:00:00 +/-0800
categories: [K8s, Monitoring]
tags: [prometheus]     # TAG names should always be lowercase
render_with_liquid: false
---

This blog is a step-by-step guide on how to deploy the kube-prometheus-stack on a Kubernetes cluster using persistent storage.

The kube-prometheus-stack is a collection of tools for monitoring Kubernetes clusters, which includes Prometheus, Alertmanager, and Grafana. By the end of this guide, you will have a fully functional monitoring system set up on your Kubernetes cluster, which will allow you to collect and analyze metrics from your cluster's resources.

The guide will cover the use of Helm, a package manager for Kubernetes, and the creation of Persistent Volumes (PV) and Persistent Volume Claims (PVC) to ensure that the data collected by the monitoring system is persistently stored.

## Step 1: Install Helm

Install Helm on your system. Helm is a package manager for Kubernetes that is used to deploy the kube-prometheus-stack.

```bash
curl https://get.helm.sh/helm-v3.4.1-linux-amd64.tar.gz -o helm.tar.gz
tar xvf helm.tar.gz
sudo mv linux-amd64/helm /usr/local/bin/
helm version
```

## Step 2: Adding Helm Charts

Add the Prometheus Community Helm repository by running the command:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

## Step 3: Updating Helm Charts

Update the local Helm repository by running the command:
```bash
helm repo update
```

## Step 4: Create K8s Namespace

Create a namespace for the kube-prometheus-stack by running the command:
```bash
kubectl create namespace monitoring
```



## Step 5: Creating PV

Create a Persistent Volume (PV) yaml file called `kube-prometheus-stack-pv.yaml` with the following content:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: kube-prometheus-stack-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/path/to/storage"
```

## Step 6: Creating PVC 

Create a Persistent Volume Claim (PVC) yaml file called `kube-prometheus-stack-pvc.yaml` with the following content:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: kube-prometheus-stack-pvc
  namespace: monitoring
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  volumeName: kube-prometheus-stack-pv
```

## Step 7: Applying PV and PVC

Apply the PV and PVC to your cluster by running the commands:
```bash
kubectl apply -f kube-prometheus-stack-pv.yaml
kubectl apply -f kube-prometheus-stack-pvc.yaml
```

## Step 8: Installing Helm Chart

Use Helm to deploy the kube-prometheus-stack in the monitoring namespace by running the command:
```bash
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack -n monitoring --set=alertmanager.persistentVolume.existingClaim=kube-prometheus-stack-pvc,server.persistentVolume.existingClaim=kube-prometheus-stack-pvc,grafana.persistentVolume.existingClaim=kube-prometheus-stack-pvc
```

## Step 9: Deployment Validation 

Verify that the kube-prometheus-stack is running by running the command:
```bash
kubectl get pods -n monitoring
```

## Step 10: Port-Forwarding Prometheus 

To access Prometheus and Grafana UI, you need to create a port-forwarding.
```bash
kubectl port-forward -n monitoring svc/kube-prometheus-stack-prometheus-operator-prometheus 9090
kubectl port-forward -n monitoring svc/kube-prometheus-stack-grafana 3000
```

This command will create port forwarding from your local machine to the Prometheus and Grafana pods running in the `monitoring` namespace. It allows you to access Prometheus and Grafana's web interfaces on your local machine by browsing `http://localhost:9090` for Prometheus and `http://localhost:3000` for Grafana.

You can open two different terminals, one for Prometheus and one for Grafana and run the above commands in parallel.

## Step 11: Accessing UI 

You can access Prometheus by browsing to `http://localhost:9090` and Grafana by browsing to `http://localhost:3000`

> Note:
Make sure to replace "/path/to/storage" with the actual path on your cluster's node where you want to store the data. Also, you may use different provisioner like NFS, GlusterFS, Ceph, etc. Make sure to have kubectl and kubeconfig setup properly.

By following these steps, you should now have the kube-prometheus-stack deployed on your Kubernetes cluster using a persistent volume for storage.
