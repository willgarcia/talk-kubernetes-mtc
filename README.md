## Content

Guide - http://aksworkshop.io/

App - https://github.com/Azure/azch-captureorder

## Deploy Kubernetes cluster

```
az group create --name mtc-event --location eastus
az aks create --name mtc-event --resource-group mtc-event --kubernetes-version 1.11.5
```

## Configure Helm

```
kubectl create -f https://raw.githubusercontent.com/Azure/helm-charts/master/docs/prerequisities/helm-rbac-config.yaml
helm init --service-account tiller
```

## Run MongoDB container

```
helm install --name orders-mongo stable/mongodb --set mongodbUsername=orders-user,mongodbPassword=orders-password,mongodbDatabase=akschallenge
```

## Run Capture API container / create service

Update manifest with team name and application insights instrumentation key

```
kubectl apply -f provision-order.yaml
```

## Submit order

```
curl -d '{"EmailAddress": "email@domain.com", "Product": "prod-1", "Total": 100}' -H "Content-Type: application/json" -X POST http://<IP>/v1/order
```

## Spam orders

```
export URL=http://<IP>/v1/order
export DURATION=1m
export CONCURRENT=300
docker run --rm -it azch/loadtest -z $DURATION -c $CONCURRENT -d '{"EmailAddress": "email@domain.com", "Product": "prod-1", "Total": 100}' -H "Content-Type: application/json" -m POST $URL
```

## Configure log streaming

```
kubectl apply -f logreader-rbac.yaml
```

