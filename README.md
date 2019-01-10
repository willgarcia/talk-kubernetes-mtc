

# Install MongoDB

```
helm install --name orders-mongo stable/mongodb --set mongodbUsername=orders-user,mongodbPassword=orders-password,mongodbDatabase=akschallenge
```

## Install Application

```
kubectl apply -f provision-order.yaml
```