# Kubeflow Pipelines Installation

[Kubeflow Pipelines](https://www.kubeflow.org/docs/pipelines/pipelines-overview/) is a platform for deploying portable, scalable machine learning workflows on Kubernetes. This document walks through the steps to install and configure Kubeflow pipelines on a private OpenStack cloud platform.
  
## (Optional) Enable GPU passthrough

The following steps walk through the configuration of GPU passthrough for a Kubernetes cluster running on OpenStack with Nvidia devices. If the Kubernetes deployment is already configured to enable GPUs please proceed to the [next section](## Install Kubeflow )

Follow platform specific mechanism for enabling GPUs on OpenStack Compute nodes. 

### Install Nvidia Docker
Follow the [Nvidia documentation](https://github.com/NVIDIA/nvidia-docker) to install and enable cuda-drivers and nvidia-container-runtime.

### Enable Nvidia Device Plugin

## Install Kubeflow 
These steps can be run on any host and not necessarily on the Kubernetes cluster where you want to run your workloads.

### Steps for Linux

```

curl -L -O https://github.com/kubeflow/kubeflow/releases/download/v0.5.0/kfctl_v0.5.0_linux.tar.gz
tar xf kfctl_v0.5.0_linux.tar.gz
sudo mv kfctl /usr/bin/
rm -f kfctl_v0.5.0_linux.tar.gz
```

### Steps for Mac

```
curl -L -O https://github.com/kubeflow/kubeflow/releases/download/v0.5.0/kfctl_v0.5.0_darwin.tar.gz (if running from Mac)
tar xf kfctl_v0.5.0_darwin.tar.gz
sudo mv kfctl /usr/bin/
rm -f kfctl_v0.5.0_darwin.tar.gz
```

## Install the Kubeflow Pipelines DSL Compiler

```
alias python=python3
pip3 install https://storage.googleapis.com/ml-pipeline/release/0.1.17/kfp.tar.gz --upgrade
```

## (Optional) Configure Kubeconfig

If running the DSL compiler and kfctl from a different host than the Kubernetes cluster, make sure you configure Kubeconfig to point to that cluster.

## Initialize and install Kubeflow Pipelines

kf-demo is the name we are giving for our sample application.
This can be replaced with any name of your choice.

```
export KFAPP=kf-demo
kfctl init ${KFAPP} --version e6a363a51a4963a624758c931b5a13e56d98d8e0
cd  ${KFAPP}
kfctl generate k8s
kfctl apply k8s
```

## (Optional) Create an NFS dynamic provisioner

Some of the steps in a Kubeflow pipeline may share common data. To facilitate sharing of data from one stage of the pipeline to the next, we need a persistent volume store that allows ReadWriteMany access. If your storage provisioner does not support ReadWriteMany, one solution is to deploy an NFS server using the ReadWriteOnce storage that is provided by the default storage provisioner. You can do it as follows:

```
helm repo update
helm install stable/nfs-server-provisioner --name kf --set=persistence.enabled=true,persistence.storageClass=standard,persistence.size=200Gi
```
Please note that the above assumes your default storage class is named standard. This is the case in GKE and some on-prem deployments. For AKS, the default storage class is named `default`, so you need to run the following instead:

```
helm repo update
helm install stable/nfs-server-provisioner --name kf --set=persistence.enabled=true,persistence.storageClass=default,persistence.size=200Gi
```

Please see [the nfs provisioner helm chart documentation](https://github.com/helm/charts/tree/master/stable/nfs-server-provisioner) for more configuration options.

### Create persistent volumes for mysql and argo

Create the Persistent Volume Claims required for mysql and argo claims.

```
kubectl -n kubeflow apply -f pvc/
```

## Deploy Kubeflow pipelines

You are ready to deploy Kubeflow Pipelines at this point. Kubeflow Pipelines comes with built in examples that you can try out to get started.
