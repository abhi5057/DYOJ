# DYOJ (Design Your Own Jewelry) Setup Instructions

This document provides instructions on how to set up the DYOJ platform. Currently, the code is in a single repository, but it should be split into multiple repositories to align with the microservices architecture.

## Repository Splitting Instructions

To move from the current single repository (`DYOJ`) to a microservices layout, follow these steps:

1. **Create New Repositories**: Create the following new repositories in your Git hosting provider (e.g., GitHub, GitLab):
    * `dyoj-ui-shell` (The host container for Microfrontends)
    * `dyoj-ui-catalog` (MFE for uploading designs and browsing)
    * `dyoj-ui-checkout` (MFE for cart and payment)
    * `dyoj-ui-dashboard` (MFE for order tracking and user profile)
    * `dyoj-api-gateway`
    * `dyoj-user-service`
    * `dyoj-catalog-service`
    * `dyoj-order-service`
    * `dyoj-payment-service`
    * `dyoj-tracking-service`
    * `dyoj-notification-service`
    * `dyoj-infra` (Optional: for docker-compose, helm charts, and global configurations)

2. **Move the Frontend**:
    * Initialize the UI repositories (`dyoj-ui-shell`, `dyoj-ui-catalog`, etc.) using Vite with the Module Federation plugin.
    * Ensure the Shell application is configured to load the distinct MFEs and manage the global Event Bus for client-side communication.

3. **Setup Backend Services**:
    * Use [Spring Initializr](https://start.spring.io/) to bootstrap each of the backend services (`dyoj-user-service`, `dyoj-order-service`, etc.) using Java 17+ and Spring Boot 3.x.
    * Ensure each service includes the necessary dependencies (Web, Data JPA, Kafka, Redis, Actuator, Micrometer).

4. **Manage Infrastructure**:
    * Move `docker-compose.yml` to the `dyoj-infra` repository (or keep it in a central documentation repo if preferred).

## Environment Configuration

Configuration is managed via distinct environment files to allow for explicit control across environments.

* `.env.dev`: Pre-populated for local development convenience.
* `.env.stage`: Template for staging environments (secrets injected by CI/CD or Vault).
* `.env.prod`: Template for production environments (secrets injected by CI/CD or Vault).

## Local Development Setup

To run the backing services (Database, Message Broker, Cache) locally:

1. Ensure Docker and Docker Compose are installed.
2. Navigate to the directory containing `docker-compose.yml`.
3. Load the development environment variables and start the containers:

    ```bash
    docker compose --env-file .env.dev up -d
    ```

4. This will start:
    * PostgreSQL on port 5432 (Databases configured per `init.sql`)
    * Redis on port 6379
    * Zookeeper on port 2181
    * Kafka on port 9092

You can now start developing and running the individual microservices against these local infrastructure components.

## Architecture Documentation

Please refer to `ARCHITECTURE.md` for a detailed breakdown of the system architecture, technology stack, data flow, and resiliency patterns.
