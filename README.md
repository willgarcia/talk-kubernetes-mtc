## Azure Kubernetes Service Challenge

This repo includes assets and commands needed to complete the AKS challenge. The goal of this challenge is to gain hands on experience with both Azure Kubernetes Service and related tech such as Azure Container Registry, Azure Monitor, and Azure Application Insights.

- Guide - http://aksworkshop.io/
- App - https://github.com/Azure/azch-captureorder

## Deploy Kubernetes cluster

First, deploy an Azure Kubernetes Cluster.

```
az group create --name mtc-workshop --location eastus
az aks create --name mtc-workshop --resource-group mtc-workshop --kubernetes-version 1.11.5 --enable-addons monitoring
```

Once completed, get the cluster credentials.

```
az aks get-credentials --name mtc-workshop --resource-group mtc-workshop --admin
```

## Configure Helm

Note, this gives Helm unrestricted access to the cluster.

```
kubectl create -f https://raw.githubusercontent.com/Azure/helm-charts/master/docs/prerequisities/helm-rbac-config.yaml
helm init --service-account tiller
```

## Run MongoDB container

Use Helm to run a container with MongoDB.

```
helm install --name orders-mongo stable/mongodb --set mongodbUsername=orders-user,mongodbPassword=orders-password,mongodbDatabase=akschallenge
```

## Run Capture API container / create service

Run an instances of the Capture API application. Update the manifest with a team name and application insights instrumentation key.

```
kubectl apply -f provision-order.yaml
```

## Submit order

To validate that the application is functional, submit an order with this command. Update the IP address with the external IP given to the applications Kubernete service.

```
curl -d '{"EmailAddress": "email@domain.com", "Product": "prod-1", "Total": 100}' -H "Content-Type: application/json" -X POST http://23.96.36.227/v1/order
```

## Spam orders

Generate traffic to the application with the following operation.

```
export URL=http://23.96.36.227/v1/order
export DURATION=1m
export CONCURRENT=300
docker run --rm -it azch/loadtest -z $DURATION -c $CONCURRENT -d '{"EmailAddress": "email@domain.com", "Product": "prod-1", "Total": 100}' -H "Content-Type: application/json" -m POST $URL
```

## Monitoring: Configure log streaming

```
kubectl apply -f logreader-rbac.yaml
```

## Scale

Create Horizontal Pod Scaling object. Shortly after the capture API pods should scale to 4.

```
kubectl create -f captureorder-hpa.yaml
```

## Create Azure Container Registry

```
az acr create --resource-group mtc-workshop --name mtcworkshop007 --sku Standard --location eastus
```

Login with the Azure CLI.

```
az acr login --name mtcworkshop007
```

Pull capture order image, tag, and push to ACR.

```
docker pull azch/captureorder
```

```
docker tag azch/captureorder mtcworkshop007.azurecr.io/captureorder
```

```
docker push mtcworkshop007.azurecr.io/captureorder
```

## Use Azure Container Registry to build image

Docker images can also be built with Azure Container Registry tasks without the need to a Docker environment. This is great when creating container image artifacts in a continuous integration / deployment solution.

To use ACR Tasks to create a Docker image, you need a Docker file and potentially the source code that will be packaged into the container image.

```
git clone https://github.com/Azure/azch-captureorder.git
cd azch-captureorder
```

The following command will create a container image and store it in your Azure Container Registry instance.

```
az acr build -t captureorder:{{.Run.ID}} -r mtcworkshop007 .
```

