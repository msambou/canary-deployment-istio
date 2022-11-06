# Canary Deployment using Istio

## Learning Objectives
At the end of this tutorial, you will learn how to;
1. Deploy multiple versions of a microservice unto a Kubernetes cluster
2. Use Istio Service Mesh to control traffic to various versions of the deployed microservice.

## Assumptions
The tutorial assumes that you have provisioned an Ubuntu 20.04 on Azure.


## Install Kuberbetes
Install the latest version of MicroK8s using the command

    sudo snap install microk8s --classic

Enable Istio with the following command:

    microk8s.enable istio

When prompted, choose whether to enforce mutual TLS authentication among sidecars. If you have a mixed deployment with non-Istio and Istio enabled services or you’re unsure, choose No.

Please run the following command to check deployment progress:

    watch microk8s.kubectl get all --all-namespaces

## Install Kubectl

## Install Istio Service Mesh

## Deploy BookInfo
Follow the instructions in [this tutorial](https://istio.io/latest/docs/examples/bookinfo/) to deploy BookInfo


