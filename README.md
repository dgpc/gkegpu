<!--
Copyright 2020 Google LLC.
SPDX-License-Identifier: Apache-2.0
-->

# GKE with GPU example application

This repository walks through deploying a proof-of-concept application on GKE that makes use of NVIDIA GPUs.

For more details, see: https://cloud.google.com/kubernetes-engine/docs/how-to/gpus

## Pre-requisites

Create a Cloud Cloud project and enable billing.

Create a VPC network (or review the "default" VPC network configuration).

## Set up instructions

Follow these instructions the first time you build and deploy the app.

```bash
# Update these variables
PROJECT_ID=dgc-playground
CLUSTLER_NAME=demo-cluster
NETWORK_NAME=default
ZONE=us-central1-c

# Set the project environment
gcloud config set project $PROJECT_ID

# Create a VPC network (skip if you want to put the nodes in an existing VPC network)
gcloud compute networks create $NETWORK_NAME --subnet-mode=auto

# Create the GKE cluster with GPU nodes
gcloud container clusters create $CLUSTER_NAME --zone $ZONE --network $NETWORK_NAME \
  --machine-type "n1-standard-1" --accelerator "type=nvidia-tesla-k80,count=1" --num-nodes "1" \
  --image-type "COS" --disk-type "pd-standard" --disk-size "100" \
  --no-enable-basic-auth --metadata disable-legacy-endpoints=true --enable-ip-alias \
  --enable-autoupgrade --enable-autorepair

# Configure kubectl to access the cluster
gcloud container clusters get-credentials $CLUSTER_NAME --zone $ZONE

# Confirm that your node is in state Ready
kubectl get nodes

# Install the NVIDIA drivers on the nodes
kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/container-engine-accelerators/master/nvidia-driver-installer/cos/daemonset-preloaded.yaml

# Confirm that nvidia-driver-installer and nvidia-gpu-device-plugin are running (this can take a few minutes)
kubectl get pods --namespace=kube-system

# Build a Docker image with the Hello GPU application and store it in your container registry
gcloud builds submit ...

# Deploy the docker image to your cluster
kubectl apply -f workload.yaml

# Confirm the pod starts up successfully, and obtain the pod name
kubectl get pods
```

Note: this will deploy your workload to the cluster, but not expose it on a public IP.
You can connect to this workload via an SSH tunnel using:

```bash
kubectl ssh ...
```

To deploy the workload publicly, use:

```bash
kubectl apply -f ingress.yaml
```

Optionally, edit `ingress.yaml` to reference a reserved public IP address to ensure your application has a consistent endpoint.

## Update instructions

Follow these instructions to deploy a new version of the Python application to your GKE cluster.

```bash
gcloud container cluster get-credentials ...
gcloud builds submit ...
kubectl ...
```

## To build and run locally

Note: running locally requires running on a machine with GPU resources available.

```bash
docker build -f Dockerfile
docker run ...
# Navigate to localhost:....
```