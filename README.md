# App Infrastructure Deployment

This repository contains Helm charts for deploying a full-stack application consisting of a React frontend, Spring Boot backend, and PostgreSQL database. It also includes monitoring with Prometheus, PostgreSQL Exporter, and Grafana.

## Project Overview

### Components Deployed with Helm
- **Frontend:** React application
- **Backend:** Spring Boot application
- **Database:** PostgreSQL
- **Monitoring Stack:**
  - Prometheus for collecting metrics
  - PostgreSQL Exporter for database metrics
  - Grafana for visualizing metrics

### Additional Components (Not in Repo)
- **Jenkins CI Pipeline:**
  - Pulled Jenkins Docker image and installed Pipeline, Maven, and Docker build plugins.
  - Used Ngrok to expose Jenkins for GitHub webhook integration.
- **ArgoCD:** Installed using the default Helm chart.
- **Nginx Ingress Controller:** Installed using the official Helm chart.

## Deployment Instructions

### Clone the Repository
```sh
git clone https://github.com/mladenovskistefan111/app-infra.git
cd app-infra
```

### Deploy the Application and Monitoring Stack
```sh
helm install postgres ./postgres-helm -n app-dev
helm install springboot ./springboot-helm -n app-dev
helm install react ./react-helm -n app-dev
helm install prometheus ./springboot-helm -n monitoring
helm install postgres-exporter ./postgres-exporter-helm -n monitoring
helm install grafana ./grafana-helm -n monitoring
```

### Set Up the CI Pipeline with Jenkins
```sh
docker pull jenkins/jenkins:lts
docker run -d --name jenkins \
  -p 8081:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts
```
Access the Jenkins web interface at `http://localhost:8081`. 

Install the necessary plugins:
   - **Maven Integration Plugin**
   - **Docker Pipeline Plugin**
   - **Pipeline Plugin**

   To install plugins, navigate to `Manage Jenkins` > `Manage Plugins` > `Available` tab, search for each plugin, select them, and click `Install without restart`.

```sh
ngrok http 8081
```
Ngrok provides a public URL for GitHub webhook integration.

### Install ArgoCD
```sh
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Install and Configure Nginx Ingress Controller
```sh
helm install my-ingress ingress-nginx/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```
After the deployment is complete, run:
```sh
minikube tunnel
kubectl get svc -n ingress-nginx
```
Grab the external IP and update `/etc/hosts`:
```sh
sudo nano /etc/hosts
```
Example:
```
127.0.0.1   localhost
10.97.123.214   react.local
10.97.123.214   springboot.local
10.97.123.214   prometheus.local
10.97.123.214   grafana.local
10.97.123.214   argocd.local
```
After all the steps are done, the application can be accessed on react.local, localy.
At argocd.local, ArgoCD can be accessed and here you can add the url of the infra repo and add all the helm charts to have CD.
At grafana.local, you can create custom Dashboards for metrics that you want to track for the application.

## Summary
- This repo contains Helm charts for deploying a React/Spring Boot/PostgreSQL application with monitoring.
- CI/CD pipeline is managed with Jenkins, exposed using Ngrok.
- ArgoCD and Nginx Ingress Controller are installed using Helm.
- The Ingress Controller requires updating `/etc/hosts` with the assigned external IP.

## Deployment Considerations
This deployment is intended for local development and testing purposes, demonstrating the integration of various DevOps tools and practices. While it serves as a valuable learning project, it may not be suitable for production environments due to several factors:

- **Security**: Secrets management is simplified and should ideally use dedicated solutions like HashiCorp Vault or AWS Secrets Manager for secure storage.
- **Networking**: The setup does not include public DNS or SSL certificates, making it inaccessible over the internet. This limits its use in real-world applications where secure, reliable access is crucial.
- **Scalability**: The deployment is designed for a local environment and may require additional configurations for scaling, resilience, and high availability in production.
Overall, this project provides a solid foundation for practicing and enhancing your DevOps skills, showcasing the deployment of a full-stack application with monitoring capabilities and CI/CD processes.


### Author
**Stefan Mladenovski** - DevOps Enthusiast

