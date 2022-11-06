# Canary Deployment using Istio

## Learning Objectives
At the end of this tutorial, you will learn how to;
1. Deploy multiple versions of a microservice unto a Kubernetes cluster
2. Use Istio Service Mesh to control traffic to various versions of the deployed microservice.

## Assumptions
The tutorial assumes that you have provisioned an Ubuntu 20.04 server on Azure.
The server should have an 8Gb RAM.


## Install Kuberbetes
Install the latest version of MicroK8s using the command

    sudo snap install microk8s --classic
    
    sudo usermod -a -G microk8s $USER
	
	sudo chown -f -R $USER ~/.kube
    
    # Reload user groups
    newgrp microk8s

Enable Istio with the following command:
    
    sudo microk8s enable community
    
    sudo microk8s.enable istio
    
    newgrp microk8s
## Download Istio Files
    
    curl -L https://istio.io/downloadIstio | sh -

    cd istio-1.15.3

### Add the istioctl client to your path 
    
    export PATH=$PWD/bin:$PATH
    
### Add a namespace label to instruct Istio to automatically inject Envoy sidecar proxies when you deploy your application later:    
    
    microk8s kubectl label namespace default istio-injection=enabled
    
## Deploy BookInfo
    
    cd istio-1.15.3
    
    sudo microk8s kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
    
## Verify Whether the Pods are running
    
    sudo microk8s kubectl get pods

## Verify Whether the Services are running
    
    sudo microk8s kubectl get services
 
    
## Open the application to outside traffic

    sudo microk8s kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
    
## 

When prompted, choose whether to enforce mutual TLS authentication among sidecars. If you have a mixed deployment with non-Istio and Istio enabled services or you’re unsure, choose No.

Please run the following command to check deployment progress:

    watch microk8s.kubectl get all --all-namespaces

## Install Kubectl

## Install Istio Service Mesh

## Deploy BookInfo
Follow the instructions in [this tutorial](https://istio.io/latest/docs/examples/bookinfo/) to deploy BookInfo


