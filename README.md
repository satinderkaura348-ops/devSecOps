# Petclinic Application on AKS with CI/CD, Docker, and HTTPS

This repository contains the **Spring Petclinic** application deployed on **Azure Kubernetes Service (AKS)** with a full **CI/CD pipeline**, Docker image management via **JFrog & Docker Hub**, and HTTPS support via **cert-manager** and **Let's Encrypt**.

---

## Table of Contents

- [Technologies Used](#technologies-used)  
- [Architecture](#architecture)  
- [CI/CD Pipeline](#cicd-pipeline)  
- [Deployment on AKS](#deployment-on-aks)  
- [Ingress and HTTPS](#ingress-and-https)  
- [DNS Configuration](#dns-configuration)  
- [Environment Variables](#environment-variables)  
- [Troubleshooting](#troubleshooting)  

---

## Technologies Used

- **Backend:** Spring Boot (Petclinic)  
- **Containerization:** Docker  
- **Artifact Repository:** JFrog Artifactory  
- **CI/CD:** GitHub Actions  
- **Kubernetes:** AKS (Azure Kubernetes Service)  
- **Ingress Controller:** NGINX  
- **TLS/SSL:** cert-manager with Let's Encrypt  
- **DNS/Domain:** devlearn.space (GoDaddy)

---

## Architecture

```text
GitHub → CI/CD → JFrog/Docker Hub → AKS → Petclinic Pods → NGINX Ingress → HTTPS (Let's Encrypt)
Code is pushed to GitHub.

GitHub Actions build the JAR via Maven and push it to JFrog Artifactory.

Docker image is built from the JAR, tagged with latest and commit SHA, and pushed to Docker Hub.

AKS pulls the Docker image and runs Petclinic pods.

NGINX Ingress exposes the application via a domain (devlearn.space).

cert-manager requests a TLS certificate from Let's Encrypt for HTTPS.

CI/CD Pipeline
Build Workflow (.github/workflows/build.yml)
Checkout code

Set up JDK 17 and cache Maven dependencies

Build project using ./mvnw clean package -DskipTests

Upload the JAR as artifact

Configure JFrog CLI and upload artifact

Docker Image Push Workflow (.github/workflows/imagepush.yml)
Download build artifact from previous workflow

Build Docker image:

bash
Copy code
docker build -t satinderkaura348/petapp:<commit-sha> -t satinderkaura348/petapp:latest .
Log in to Docker Hub

Push both latest and commit SHA tagged images

Deployment on AKS
Set your Azure subscription:

bash
Copy code
az account set --subscription <your-subscription-id>
Get AKS credentials:

bash
Copy code
az aks get-credentials --resource-group my_rg --name myAKSCluster --overwrite-existing
Deploy the Petclinic service and deployment:

bash
Copy code
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
Check pod status:

bash
Copy code
kubectl get pods -o wide
kubectl logs <pod-name>
Update image after new CI/CD push:

bash
Copy code
kubectl set image deployment/petclinic petclinic=satinderkaura348/petapp:latest
kubectl rollout status deployment/petclinic
Ingress and HTTPS
ClusterIssuer (clusterissuer.yaml)
yaml
Copy code
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    email: satinder42020@gmail.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
Ingress (ingress.yaml)
yaml
Copy code
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: petclinic-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  rules:
  - host: devlearn.space
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: petclinic-service
            port:
              number: 80
  tls:
  - hosts:
    - devlearn.space
    secretName: petclinic-tls
Once applied, cert-manager will automatically request a TLS certificate, making https://devlearn.space secure.

DNS Configuration
Domain: devlearn.space

Type: A Record

Name: @

Value: <AKS LoadBalancer IP>

TTL: 600 seconds (10 minutes)

Optionally, add www as a CNAME pointing to devlearn.space.

Environment Variables
SPRING_PROFILES_ACTIVE=dev → Spring profile

MYSQL_URL=jdbc:mysql://petclinic-mysql:3306/petclinic → MySQL connection string

Troubleshooting
CrashLoopBackOff / Unable to access jarfile:

Ensure Dockerfile copies the JAR correctly. Example:

dockerfile
Copy code
FROM eclipse-temurin:17-jdk
WORKDIR /home/petclinic/
COPY ./target/*.jar app.jar
EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
ImagePullBackOff:

Check Docker Hub for latest image. Use imagePullPolicy: Always.

Ingress not accessible:

Make sure DNS A record points to the correct LoadBalancer IP.

Ensure NGINX Ingress controller is running:

bash
Copy code
kubectl get pods -n ingress-nginx
Author
Satinder Kaura






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
