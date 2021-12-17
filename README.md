# Setup
First we will need the Azure resources

### Terraform

Login to Azure
```bash
az login
```

Deploy ADLS Gen 2 storage and Azure Kubernetes Service with the following commands:
```bash
terraform init
terraform plan -out terraform.plan
terraform apply terraform.plan
```

Go to ADLS Gen 2 resource and set the OAuth 2.0 access with an Azure service principal: https://docs.microsoft.com/en-us/azure/databricks/data/data-sources/azure/adls-gen2/azure-datalake-gen2-sp-access

Copy the following credentials:
* "abfss://container-name@storage-account.dfs.core.windows.net/" to `STORAGE_PATH` in `main.py`
* application-client-id to `APP_ID` in `main.py`
* directory-id to `DIRECTORY_ID` in `main.py`
* client-secret to `AZURE_SECRET` in `src/.env` file (see `src/.env.example`)


### Geocoding API

Sign up on https://opencagedata.com/ , get the API key and put it in the `src/.env` file 

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

# Run

Submit the Spark Job with the following command:
```bash
spark-submit --master k8s://<your-azure-kubernetes-cluster>.hcp.<region>.azmk8s.io\
 --deploy-mode cluster --name m06sparkbasics --executor-memory 3G --driver-memory 3G\
  --conf spark.sql.broadcastTimeout=6000 --conf spark.kubernetes.container.image.pullPolicy=Always\
   --conf spark.kubernetes.container.image=<your-azure-container-registry>.azurecr.io/<container-name>:<tag>\
     local:///opt/main.py

```

# Result

Pods started

![pods_started](screenshots/pods_started.png)

Spark UI view:

![spark_ui_view](screenshots/spark_ui_view.png)


The job got completed successfully

![pods_completed](screenshots/pods_completed.png)

We can see the result data in ADLS Gen 2 with preserved partitioning

![results](screenshots/results.png)

Example of joined data

![result_example](screenshots/result_example.png)

