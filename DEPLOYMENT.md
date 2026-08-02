# Deployment Architecture & Cloud Strategy

This document outlines the production deployment strategy for the DYOJ platform. While local development uses `docker-compose`, production deployment will leverage hyperscaler cloud services for maximum scalability, resilience, and operational efficiency.

## 1. Cloud Provider Selection

### Primary Provider: Amazon Web Services (AWS)

AWS is selected due to its mature ecosystem, vast global network (important for low-latency MFE delivery and global order tracking), and best-in-class managed services for our chosen tech stack (Kubernetes, Kafka, Redis, Postgres).

*> (Alternative: Google Cloud Platform (GCP) offers comparable services (GKE, Cloud SQL, Memorystore, Pub/Sub), but AWS MSK provides a more direct translation from our local Confluent Kafka setup without vendor lock-in.)

## 2. Infrastructure as Code (IaC)

All cloud infrastructure will be provisioned using **Terraform**. This ensures the infrastructure is version-controlled, reproducible, and easily deployable across environments (Staging, Production).

* The Terraform state will be stored securely in an S3 bucket with DynamoDB state locking.
* Secrets (like database passwords and Stripe API keys) will be managed via **AWS Secrets Manager**, avoiding any `.env` files in production.

## 3. Deployment Components

### A. Compute Layer (Microservices)

* **Platform:** **Amazon EKS (Elastic Kubernetes Service)**
* **Strategy:** Spring Boot microservices (`dyoj-user-service`, `dyoj-order-service`, etc.) will be containerized using Docker and deployed as Kubernetes Pods.
* **Scaling:** EKS Cluster Autoscaler and Horizontal Pod Autoscalers (HPA) will dynamically scale the number of pods based on CPU/Memory utilization and custom Actuator metrics (e.g., scaling up `dyoj-catalog-service` during high image upload traffic).
* **Networking:** An **AWS Application Load Balancer (ALB)** via the AWS Load Balancer Controller will route incoming public traffic to the `dyoj-api-gateway` pod. Internal microservice-to-microservice communication will happen via Kubernetes CoreDNS within the private VPC.

### B. Event Streaming (Kafka)

* **Platform:** **Amazon MSK (Managed Streaming for Apache Kafka)**
* **Strategy:** MSK provides a fully managed, highly available Kafka cluster. This eliminates the operational overhead of managing Zookeeper and Kafka brokers manually. It seamlessly supports the Saga pattern, transactional outbox events, and traceId propagation defined in the architecture.

### C. Caching & Realtime Backplane (Redis)

* **Platform:** **Amazon ElastiCache for Redis**
* **Strategy:** Deployed in a multi-AZ cluster with automatic failover. It serves dual purposes:
    1. Caching layer for Idempotency keys and rate-limiting.
    2. Pub/Sub fanout backplane for the `dyoj-notification-service` to distribute WebSocket events across multiple EKS pods.

### D. Database Layer (PostgreSQL)

* **Platform:** **Amazon RDS for PostgreSQL**
* **Strategy:** A highly available, Multi-AZ RDS instance. To maintain the **Database-per-Service** pattern, multiple logical databases (`dyoj_user_db`, `dyoj_order_db`, etc.) will be provisioned within the same RDS cluster, with separate IAM database authentication roles for each microservice to enforce strict isolation.

### E. Frontend Hosting (Microfrontends)

* **Platform:** **Amazon S3 + Amazon CloudFront (CDN)**
* **Strategy:**
    1. The built static assets for the React Microfrontends (`dyoj-ui-shell`, `dyoj-ui-catalog`, etc.) will be uploaded to private S3 buckets.
    2. CloudFront will serve as the global CDN, caching assets at the edge for incredibly low-latency loading worldwide.
    3. CloudFront will use Origin Access Control (OAC) to securely fetch files from S3.
    4. The shell MFE will dynamically load the remote MFEs at runtime via CloudFront URLs.

## 4. CI/CD Pipeline Flow

We will utilize **GitHub Actions** (or AWS CodePipeline) to automate deployments.

**Backend Flow:**

1. Developer merges PR to `main`.
2. GitHub Actions runs unit/integration tests.
3. Builds the Docker image and pushes it to **Amazon ECR (Elastic Container Registry)**.
4. Updates the Kubernetes Helm chart with the new image tag.
5. Applies the Helm chart to the EKS cluster.

**Frontend Flow:**

1. Developer merges PR to `main` in an MFE repository.
2. GitHub Actions runs Vite build.
3. Syncs the resulting `dist/` folder to the corresponding S3 bucket.
4. Invalidates the CloudFront cache to serve the latest UI immediately.
