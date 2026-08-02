# DYOJ (Design Your Own Jewelry) Setup Instructions

This document provides instructions on how to set up the DYOJ platform. Currently, the code is in a single repository, but it should be split into multiple repositories to align with the microservices architecture.

## Repository Splitting Instructions

To move from the current single repository (`DYOJ`) to a microservices layout, follow these steps:

1. **Create New Repositories**: Create the following new repositories in your Git hosting provider (e.g., GitHub, GitLab):
    * `dyoj-ui`
    * `dyoj-api-gateway`
    * `dyoj-user-service`
    * `dyoj-catalog-service`
    * `dyoj-order-service`
    * `dyoj-payment-service`
    * `dyoj-tracking-service`
    * `dyoj-notification-service`
    * `dyoj-infra` (Optional: for docker-compose, helm charts, and global configurations)

2. **Move the Frontend**:
    * Initialize `dyoj-ui` using Vite or Create React App.
    * Move any existing UI prototype files (e.g., `DYOJ Prototype.dc.html`, `image-slot.js`, `support.js`, `_ds`, `design_handoff_dyoj`) into a `docs/` or `prototypes/` folder within `dyoj-ui` for reference.

3. **Setup Backend Services**:
    * Use [Spring Initializr](https://start.spring.io/) to bootstrap each of the backend services (`dyoj-user-service`, `dyoj-order-service`, etc.) using Java 17+ and Spring Boot 3.x.
    * Ensure each service includes the necessary dependencies (Web, Data JPA, Kafka, Redis, Actuator, Micrometer).

4. **Manage Infrastructure**:
    * Move `docker-compose.yml` to the `dyoj-infra` repository (or keep it in a central documentation repo if preferred).

## Local Development Setup

To run the backing services (Database, Message Broker, Cache) locally:

1. Ensure Docker and Docker Compose are installed.
2. Navigate to the directory containing `docker-compose.yml`.
3. Run the following command:

    ```bash
    docker compose up -d
    ```

4. This will start:
    * PostgreSQL on port 5432
    * Redis on port 6379
    * Zookeeper on port 2181
    * Kafka on port 9092

You can now start developing and running the individual microservices against these local infrastructure components.

## Architecture Documentation

Please refer to `ARCHITECTURE.md` for a detailed breakdown of the system architecture, technology stack, data flow, and resiliency patterns.
