

**🚀 Deploying to Azure Kubernetes Service (AKS)**

This guide explains how to set up Azure resources and GitHub Actions so your application can be automatically deployed to AKS using Helm.

1️⃣ Create a Resource Group

The resource group will contain your AKS cluster and related resources.

```
az group create \
  --name <RESOURCE_GROUP_NAME> \
  --location <AZURE_REGION>
```

2️⃣ Create an AKS Cluster

This creates a single-node AKS cluster with monitoring enabled.

```
az aks create \
  --resource-group <RESOURCE_GROUP_NAME> \
  --name myAKSCluster \
  --node-count 1 \
  --enable-addons monitoring \
  --generate-ssh-keys \
  --kubernetes-version 1.28.0
```

3️⃣ (Optional) Connect to the Cluster Locally

Use this only if you want to test using kubectl from your own machine.
GitHub Actions does NOT need this.

```
az aks get-credentials \
  --resource-group <RESOURCE_GROUP_NAME> \
  --name myAKSCluster \
  --overwrite-existing
```

4️⃣ Create a Service Principal for GitHub Actions

GitHub needs permissions to authenticate to Azure and deploy your app.
This command creates a Service Principal in the correct JSON format expected by azure/login@v1.

```
az ad sp create-for-rbac \
  --name github-aks-sp \
  --role contributor \
  --scopes /subscriptions/<SUBSCRIPTION_ID> \
  --sdk-auth
```

This will output a JSON block containing:

clientId

clientSecret

subscriptionId

tenantId

Azure endpoints

Copy the entire JSON output and save it in GitHub Secrets as:

```
AZURE_CREDENTIALS
```
