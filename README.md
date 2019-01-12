## Content

Guide - http://aksworkshop.io/
App - https://github.com/Azure/azch-captureorder

## Pre RBAC for log streeming

```
kubectl apply -f logreader-rbac.yaml
```

## Install MongoDB

```
helm install --name orders-mongo stable/mongodb --set mongodbUsername=orders-user,mongodbPassword=orders-password,mongodbDatabase=akschallenge
```

## Install Application

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

