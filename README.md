# 3-Tier-application-backend-Database-proxy-
3-Tier application (backend,Database,proxy)  from kubernetes to openshift

3-Tier Go Web Application for OpenShift
This repository contains the manifests and documentation for deploying a 3-tier web application (Nginx, Go, MySQL) on an OpenShift cluster.

The primary purpose of this project is to demonstrate the migration from a standard, insecure Kubernetes configuration to a secure, production-ready OpenShift deployment. It addresses OpenShift-specific challenges related to security, networking, and storage.

All resources are configured for the namespace lab16-sehs.

🏗️ Application Architecture
The application consists of three main components:

Frontend (Proxy): An Nginx container (3booda24/nginx-proxy-go:latest) that acts as the public-facing entry point, handling SSL termination and reverse-proxying requests.

Backend (App): A Go application (3booda24/app-go:latest) that contains the core business logic and connects to the database.

Database (DB): A MySQL 8.0 container that provides persistent data storage.

This diagram shows the final, secure OpenShift architecture, where an external Route directs traffic to the Nginx Service, which in turn communicates with the Nginx Pods on a non-privileged port (8443).

🔐 The Scenario: From Insecure Kubernetes to Secure OpenShift
The "base" manifests for this project were written for a standard, permissive Kubernetes cluster and failed to deploy on OpenShift. The following key challenges were identified and solved using OpenShift best practices.

Summary of Key Changes
Permissions: Moved from a cluster-wide permission (oc adm policy add-scc-to-user anyuid -z default) to a dedicated ServiceAccount (allow-anyuid). This ServiceAccount is granted the anyuid SCC and is only used by the database and proxy deployments, following the Principle of Least Privilege.

Networking & SSL: Replaced the insecure NodePort service with a ClusterIP service and a proper OpenShift edge termination Route. The Route handles the hostname and SSL termination.

Nginx & Privileged Ports: Since the anyuid SCC allows the container to run as its non-root nginx user, it can no longer bind to the privileged port 443. The container was re-configured to listen on port 8443 instead.

Custom Configuration: A ConfigMap was created to inject the custom nginx.conf (listening on 8443), and a Secret was created to inject the SSL certificates. volumeMounts were added to the deployment to use these.

Storage: The insecure hostPath PersistentVolume was deleted. The PersistentVolumeClaim was modified to use the cluster's default StorageClass for dynamic provisioning.

Image Policy: The imagePullPolicy: Never (which caused errors) was changed to imagePullPolicy: IfNotPresent to ensure the images are pulled from the registry.
