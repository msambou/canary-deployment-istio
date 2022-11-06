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
    
## Verify whether Everything works fine
	microk8s kubectl exec "$(microk8s kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"
<title>Simple Bookstore App</title>


## Open the application to outside traffic
	
	microk8s kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml

## Determining the ingress IP and ports

	microk8s kubectl get svc istio-ingressgateway -n istio-system
	

## Set the ingress IP and ports

	export INGRESS_PORT=$(microk8s kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
	
	export SECURE_INGRESS_PORT=$(microk8s kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')

	export INGRESS_HOST=$(microk8s kubectl get po -l istio=ingressgateway -n istio-system -o jsonpath='{.items[0].status.hostIP}')


## Set Gateway URL

	export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
	
	echo $GATEWAY_URL
