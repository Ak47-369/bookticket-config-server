# BookTicket :: Platform :: Config Server

## Overview

The **Config Server** is a foundational component of the BookTicket microservices ecosystem. It provides centralized, externalized configuration management for all services. By acting as a single source of truth, it ensures that all microservices run with consistent and correct properties across different deployment environments.

This service uses Spring Cloud Config Server.

## Core Responsibilities

-   **Centralized Configuration:** Serves configuration properties to all other microservices from a central location.
-   **Environment Abstraction:** Manages different configuration profiles (e.g., `dev`, `prod`) for each service.
-   **Versioning and Auditing:** Leverages a Git backend to provide a complete history of all configuration changes.

## Architecture


<img width="1119" height="784" alt="Config-Server-Architecture" src="https://github.com/user-attachments/assets/2809fa3c-ad5d-44b0-81ad-4f59fe0826af" />

```mermaid
---
config:
  theme: redux
---
flowchart TB
 subgraph subGraph0["Cloud Environment (Render)"]
        C("Config Server")
        A["API Gateway"]
        B["User Service"]
        D["Other Services"]
  end
 subgraph Infrastructure["Infrastructure"]
        E["Git Repository (central-config-server)"]
  end
    A --> C
    B --> C
    D --> C
    C --> Infrastructure

    style C fill:#f9f,stroke:#333,stroke-width:2px
    style E fill:#ccf,stroke:#333,stroke-width:2px
```

### How It Works

1.  **Backend Storage:** All configuration files (`.yaml`) are stored in a dedicated Git repository. This provides version control, history, and an auditable trail for every configuration change.
2.  **Client Bootstrap:** When a microservice (e.g., `user-service`) starts up, it makes a request to the Config Server, identifying itself by its application name and active profile (e.g., `user-service`, `prod`).
3.  **Serving Configuration:** The Config Server reads the appropriate files from the Git repository (`application.yaml` and `user-service-prod.yaml`), combines the properties, and sends them back to the client service.
4.  **Standalone Operation:** The Config Server is a foundational service and does **not** register with the Eureka Service Registry. This is a deliberate design choice to prevent circular dependencies during application startup.

## Configuration

The server is configured in its `application.yaml` file. Key properties include:

-   `spring.cloud.config.server.git.uri`: The URI of the Git repository containing the configuration files.
-   `server.port`: The port on which the Config Server listens (default: `8888`).
-   `eureka.client.enabled: false`: Explicitly disables Eureka client behavior.
-   `spring.cloud.config.enabled: false`: Prevents the server from trying to connect to itself for configuration.

## Endpoints

The Config Server exposes a REST API for clients to fetch their configuration. You can inspect the configuration for a service using the following URL structure:

`/{application}/{profile}`

-   **Example:** To see the production configuration for the `api-gateway`, you would access:
    `https://config-server-f2t3.onrender.com/api-gateway/prod`
