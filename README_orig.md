# Kafka connect in Kubernetes

## Install Confluent Hub Client

You can find the installation manual [here](https://docs.confluent.io/home/connect/confluent-hub/client.html)

## Create a custom docker image

### Azure Container Registry

Create your Azure Container Registry https://docs.microsoft.com/en-us/azure/container-registry/container-registry-get-started-portal

Login
```bash
az acr login --name <registry-name>
```

Build your image and push it
```bash
docker build -f ./connectors/Dockerfile -t <registry-name>.azurecr.io/<image-name>:<tag> .
docker push <registry-name>.azurecr.io/<image-name>:<tag>
```

Attach the Azure Container Registry to Azure Kubernetes Service
```bash
az aks update -n <aks_resource_name> -g <aks_resource_group_name> --attach-acr <registry-name>
```

Get credentials for AKS cluster
```bash
az aks update -n <aks_resource_name> -g <aks_resource_group_name> --attach-acr <registry-name>
```


## Launch Confluent for Kubernetes

### Create a namespace

- Create the namespace to use:

  ```cmd
  kubectl create namespace confluent
  ```

- Set this namespace to default for your Kubernetes context:

  ```cmd
  kubectl config set-context --current --namespace confluent
  ```

### Install Confluent for Kubernetes

- Add the Confluent for Kubernetes Helm repository:

  ```cmd
  helm repo add confluentinc https://packages.confluent.io/helm
  helm repo update
  ```

- Install Confluent for Kubernetes:

  ```cmd
  helm upgrade --install confluent-operator confluentinc/confluent-for-kubernetes
  ```

### Install Confluent Platform

- Install all Confluent Platform components:

  ```cmd
  kubectl apply -f ./confluent-platform.yaml
  ```

- Install a sample producer app and topic:

  ```cmd
  kubectl apply -f ./producer-app-data.yaml
  ```

- Check that everything is deployed:

  ```cmd
  kubectl get pods -o wide 
  ```

### View Control Center

- Set up port forwarding to Control Center web UI from local machine:

  ```cmd
  kubectl port-forward controlcenter-0 9021:9021
  ```

- Browse to Control Center: [http://localhost:9021](http://localhost:9021)

## Create a kafka topic

- The topic should have at least 3 partitions because the azure blob storage has 3 partitions. Name the new topic: "expedia".

## Prepare the azure connector configuration

## Upload the connector file through the API
