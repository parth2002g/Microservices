
# Spring Boot Microservices Deployment to Kubernetes with Helm via GitLab CI

## Introduction
In today's fast-paced software development environment, it is essential to deploy applications quickly and efficiently. Microservices architecture is becoming increasingly popular for its ability to create highly scalable and flexible applications. Kubernetes, a popular open-source container orchestration system, has also become the go-to solution for microservices deployment. This article will walk through the process of deploying applications from microservices to a Kubernetes cluster using Helm via GitLab CI.

GitLab CI is an integral part of the GitLab platform and provides a robust continuous integration and deployment (CI/CD) pipeline. It allows developers to automate the building, testing, and deployment of their applications. Helm is a package manager for Kubernetes that allows users to install, upgrade, and manage applications in a Kubernetes cluster.

## Prerequisites
To perform this demo, you will need the following prerequisites:
- **A Kubernetes cluster**: You can use an existing Kubernetes cluster or create a new one on a cloud provider such as Google Cloud, Amazon Web Services, or Microsoft Azure.
- **A GitLab account**: Required to set up the CI/CD pipeline.
- **Docker Hub account**: Required to store Docker images.

For more technical concepts, see another published article on deploying a Spring Boot App with Docker in a Kubernetes Cluster.

## What is Helm?
Helm is a package manager for Kubernetes that simplifies the installation and management of applications in a Kubernetes cluster. Helm uses charts to define the structure and configuration of an application, which can be versioned and shared between teams.

A Helm chart is a package that contains all the Kubernetes manifests, configuration files, and dependencies needed to install and run an application on a Kubernetes cluster. It defines environment variables, ports, volumes, and service dependencies.

**Advantages of using Helm for deployment on a Kubernetes cluster:**
- **Simplified deployment**: Helm provides an efficient way to package and deploy applications, reducing complexity for developers and operators.
- **Version control**: Helm charts allow for easy rollback to previous versions in case of issues.
- **Consistency**: Automates manual tasks, reducing the risk of errors.
- **Template engine**: Helm's template engine generates Kubernetes manifests dynamically, allowing customization for different environments.

## Step 1: Create Your Own Repository
Create two repositories: one for storing the source code of the microservices and another for externalizing configuration files using **Spring Cloud Config Server**. The code can be found in the following repository:

Let's briefly highlight the importance of the folders in that repository:
- **.gitlab/agents/k8s-cluster**: Contains the configurations of the Kubernetes agent for the server.
- **ecommerce-api-gateway**: Code for the API gateway, serving as an entry point for client requests.
- **ecommerce-config-server**: Code for the configuration server, centralizing configuration information for the microservices.
- **ecommerce-order-service**: Code for the order-handling microservice.
- **ecommerce-product-service**: Code for the product-handling microservice.
- **ecommerce-sale-service**: Code for the sales-handling microservice.
- **ecommerce-service-registry**: Code for the service registry, tracking available microservices.
- **ecommerce-user-service**: Code for the user-handling microservice.
- **helm**: Files to deploy the application under Kubernetes via Helm.
- **.gitlab-ci.yml**: The core pipeline file, describing the steps of the CI/CD process.
- **docker-compose.yml**: Allows deploying the application under Docker.
- **microservices-configuration**: Should be stored in a separate repository, containing configuration files for different microservices.

## Step 2: Configuration of GitLab CI
Open the `.gitlab-ci.yml` file located at the root of the repository.

```yaml
variables:
  GATEWAY_IMAGE_NAME: yourdockerhub/api-gateway
  CONFIG_IMAGE_NAME: yourdockerhub/config-server
  ORDER_IMAGE_NAME: yourdockerhub/order-service
  PRODUCT_IMAGE_NAME: yourdockerhub/product-service
  SALE_IMAGE_NAME: yourdockerhub/sale-service
  REGISTRY_IMAGE_NAME: yourdockerhub/registry-service
  USER_IMAGE_NAME: yourdockerhub/user-service
 
stages:
  - build_push_image
  - deploy

build_push_microservice_image:
  stage: build_push_image
  image: docker:20.10.16
  services:
    - docker:20.10.16-dind
  variables:
    DOCKER_TLS_CERTDIR: "/certs"
  before_script:
    - echo $DOCKER_PASSWORD | docker login -u $DOCKER_LOGIN --password-stdin
  script: 
    - docker build -t $GATEWAY_IMAGE_NAME ecommerce-api-gateway/.
    - docker push $GATEWAY_IMAGE_NAME
    - docker build -t $CONFIG_IMAGE_NAME ecommerce-config-server/.
    - docker push $CONFIG_IMAGE_NAME
    - docker build -t $ORDER_IMAGE_NAME ecommerce-product-service/.
    - docker push $ORDER_IMAGE_NAME
    - docker build -t $PRODUCT_IMAGE_NAME ecommerce-product-service/.
    - docker push $PRODUCT_IMAGE_NAME
    - docker build -t $SALE_IMAGE_NAME ecommerce-sale-service/.
    - docker push $SALE_IMAGE_NAME
    - docker build -t $REGISTRY_IMAGE_NAME ecommerce-service-registry/.
    - docker push $REGISTRY_IMAGE_NAME
    - docker build -t $USER_IMAGE_NAME ecommerce-user-service/.
    - docker push $USER_IMAGE_NAME

deploy_application:
  stage: deploy
  image: devth/helm:latest
  before_script:
    - cd helm/
    - kubectl config get-contexts
    - kubectl config use-context your-k8s-cluster
  script: 
    - helm install mysqldb mysql
    - helm install --set-string GIT_URL_CONFIG=$GIT_URL_CONFIG config config-server
    - helm install --set-string SERVICE_REGISTRY=$SERVICE_REGISTRY registry registry-service
    - helm install --set-string CONFIG_SERVER=$CONFIG_SERVER gateway api-gateway
    - helm install --set-string CONFIG_SERVER=$CONFIG_SERVER order order-service
    - helm install --set-string CONFIG_SERVER=$CONFIG_SERVER product product-service
    - helm install --set-string CONFIG_SERVER=$CONFIG_SERVER sale sale-service
    - helm install --set-string CONFIG_SERVER=$CONFIG_SERVER user user-service
```

The `.gitlab-ci.yml` file includes:
1. **Docker image build and push**: Builds Docker images for microservices and pushes them to Docker Hub.
2. **Deploy step**: Deploys the application to Kubernetes using Helm.

### Build & Push Stage
In this step, Docker images for microservices are built and pushed to Docker Hub. You need to define `DOCKER_PASSWORD` and `DOCKER_LOGIN` in your GitLab project settings to allow GitLab CI to authenticate with Docker Hub.

### Deploy Stage
This stage uses Helm to install and configure the microservices on the Kubernetes cluster. The necessary environment variables such as `SERVICE_REGISTRY`, `CONFIG_SERVER`, and `GIT_URL_CONFIG` must be defined in GitLab.

## Step 3: Launch the Pipeline
Run the pipeline from the GitLab project repository and verify that it builds and deploys the application to Kubernetes successfully.

## Conclusion
This guide covers deploying Spring Boot microservices to Kubernetes using Helm and GitLab CI. These tools allow for efficient automation, scalability, and management of microservices in a Kubernetes cluster, streamlining the continuous integration and deployment process.