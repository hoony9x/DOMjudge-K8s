# DOMjudge with K8s

This repository contains how to run DOMjudge with Kubernetes and Docker Images.  
(Please note that this project is for my practice purpose, but it can be used for real contest...?)

## Prerequisites

- Kubernetes Cluster
  - You can use your own cluster, or
  - Managed cluster like EKS(AWS Managed K8s), GKE(GCP Managed K8s)
  
## Before start

- For MySQL, these scripts are intended to run MySQL as **Single Pod**.
  - I'll fix these scripts to make it possible to run as multiple MySQL pods later.
  - Maybe `StatefulSet` will be needed.
- For Redis, these scripts are intended to run Redis as **Single Pod**
- For DOMServer and JudgeHost, these are intended to run with **Multiple Pods**.

## How to start

Please type below commands in this order.

### MySQL

1. `$ kubectl apply -f mysql-pv.yaml` (Create Persistant Volume for MySQL)
  - If K8s is running on GCP, you don't need to type this. GCP will automatically allocate space for PV claim.
2. `$ kubectl apply -f mysql-pv-claim.yaml` (Request PV Claim for MySQL)
3. `$ kubectl apply -f mysql-secrets.yaml` (Create passwords for MySQL)
  - Password in this script is randomly generated and **encoded with base64**.
  - You need to use your own password.
  - To change password, **you need to encode your password with base64** and write into this script.
4. `$ kubectl apply -f mysql-deployment.yaml` (Create MySQL deployment)
  - It creates MySQL deployment using secrets and PV which were created with previous step.
5. `$ kubectl apply -f mysql-service.yaml` (Expose MySQL deployment)
  - It exposes MySQL deployment as `ClusterIP` so that DOMServer can access to it.

### Redis

It will be used as a session storage for DOMjudge Web Server (DOMServer)

1. `$ kubectl apply -f redis-deployment.yaml` (Create Redis deployment)
2. `$ kubectl apply -f redis-service.yaml` (Expose Redis deployment)
  - It exposes Redis deployment as `ClusterIP` so that DOMserver can access to it.

### DOMjudge WebServer (DOMServer)

1. `$ kubectl apply -f domserver-configmap.yaml` (Add configmap for DOMServer)
  - It creates configmap for DOMServer to use Redis as session storage.
2. `$ kubectl apply -f domjudge-tls-secret.yaml` (Add TLS(SSL) certificate and private key)
  - It creates secrets for TLS that will be used in `Ingress`.
  - In this script, certificate and private key were generated as self-signed with OpenSSL.
  - If you want to use your own (like, generated with Let's Encrypt), you have to encode yours with base64 too.
3. `$ kubectl apply -f domserver-deployment.yaml` (Create deployment for DOMServer)
  - It creates DOMServer deployment.
    - First, it installs `php-redis` in corresponding pod.
    - Second, it changes php.ini to enable DOMServer can use Redis as session storage.
    - Finally, run DOMServer.
  - It uses previously created MySQL, Redis services.
4. `$ kubectl apply -f domserver-service.yaml` (Expose DOMServer deployment)
  - It exposes DOMServer deployment as `NodePort` so that clients from the outside of K8s cluster can access to it.
  - Access from external clients will be handled via `Ingress`.
5. `$ kubectl apply -f domserver-ingress.yaml` (Create Ingress for DOMServer service)
  - It creates `Ingress` to make it possible to access to DOMServer service.
    - With specific port.
    - With TLS certificate (so that we can connect to DOMServer via HTTPS)
  - It runs as a Load Balancer.
  - If you use managed K8s, `Ingress` will be create automatically without any other setup.
  - If you use your own K8s cluster, maybe you need additional setup to use `Ingress`.

## TODO

- Revise each YAML files.
- Check if there is any wrong discriptions.
- Add scripts for DOMjudge JudgeHost. (Currently, it only contains scripts for MySQL, Redis and DOMServer.)
