# Canary Deployment using Istio

## Learning Objectives
At the end of this tutorial, you will learn how to;
1. Deploy multiple versions of a microservice unto a Kubernetes cluster
2. Use Istio Service Mesh to control traffic to various versions of the deployed microservice.

## Assumptions
The tutorial assumes that you have provisioned an Ubuntu 20.04 server on Azure.
The server should have an 8Gb RAM.

## Canary Deployment
With canary deployment, DevOps Engineers roll out a new version of an application only to a selected group of users.
The selected group of users may test and provide feedback to the development team. The update is later rolled out to the remaining 
group of users once the team is confident the release won't cause issues.

## BookInfo Application
In this tutorial we will demonstrate canary deployment using [BookInfo Application](https://istio.io/latest/docs/examples/bookinfo/).
The Bookinfo application is broken into four separate microservices:

* **productpage.** The productpage microservice calls the details and reviews microservices to populate the page.
* **details.** The details microservice contains book information.
* **reviews.** The reviews microservice contains book reviews. It also calls the ratings microservice.
* **ratings.** The ratings microservice contains book ranking information that accompanies a book review.


There are 3 versions of the reviews microservice:

* **Version v1** doesn’t call the ratings service.
* **Version v2** calls the ratings service, and displays each rating as 1 to 5 black stars.
* **Version v3** calls the ratings service, and displays each rating as 1 to 5 red stars.

![BookInfo Architecture](https://istio.io/latest/docs/examples/bookinfo/noistio.svg)

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
    
    sudo microk8s kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
    
## Verify Whether the Pods are running
    
    sudo microk8s kubectl get pods

## Verify Whether the Services are running
    
    sudo microk8s kubectl get services
 
    
## Verify whether Everything works fine
	microk8s kubectl exec "$(microk8s kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"
<title>Simple Bookstore App</title>


## Open the application to outside traffic
	
	microk8s kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml

## Determining the ingress IP and ports

	microk8s kubectl get svc istio-ingressgateway -n istio-system
	

## Set the ingress IP and ports

Get INGRESS_PORT

	export INGRESS_PORT=$(microk8s kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')

Set the SECURE_INGRESS_PORT

	export SECURE_INGRESS_PORT=$(microk8s kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')



## Set Gateway URL
Because our application is running on a remote server, Azure, we have to use the public IP address of the server we provisioned.
	
	export INGRESS_HOST=<your-vm-IP>
	export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
	
	echo $GATEWAY_URL

## Open all inbound ports on Azure
Now, we want to make sure that our application is available to the public, to do this we have to open certain ports on the VM we provisioned.
We will only open the INGRESS_PORT for security purposes.

Navigate to the VM created. Under **Settings** , select **Networking**
Add a new inbound rule. Destination port should be your INGRESS_PORT. Finally, click on Add.

### View Application
echo $GATEWAY_URL should give you the IP address and Port your application is running on.
In your browser, navigate to htt://$GATEWAY_URL/productpage. You should find your application running.

## Apply default routing rules

	sudo microk8s kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
	
## Create Routing Rules

	nano myMeshConfig.yaml

### Copy and Paste the following into the myMeshConfig.yaml

	apiVersion: networking.istio.io/v1alpha3
	kind: VirtualService
	metadata:
	  name: reviews
	spec:
	  hosts:
	  - reviews
	  http:
	  - name: "BookInfo Custom Routing"
	    route:
	    - destination:
		host: reviews
		subset: v1
	      weight: 70
	    - destination:
		host: reviews
		subset: v2
	      weight: 20
	    - destination:
		host: reviews
		subset: v3
	      weight: 10   
      



Save and close the editor.

In the above configuration, we are sending 70% of the traffic to version 1, 20% to version 2 and 10% to version 3 of the review application.

### Apply these routing rules

	sudo microk8s kubectl apply -f myMeshConfig

### View the updates in your browser

	- V1 No ratings/stars
	- V2 Rating with Black stars
	- V3 Rating with Red stars
	
