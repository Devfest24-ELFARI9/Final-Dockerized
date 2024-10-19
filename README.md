# Sensor-Data-Ingestion-Service

To clone all file use the following clone command
```
git clone --recurse-submodules git@github.com:Devfest24-ELFARI9/Final-Dockerized.git
```



Make sure to migrate

```
dokcer exec -it devfestproject_dashboard_1 npx prisma migrate dev
```



# Sensor Data Ingestion and Monitoring System

## Project Overview

This project implements a sensor data ingestion and processing system using a microservices architecture. The architecture is designed for real-time monitoring and alerting in a manufacturing environment, using **RabbitMQ** as a message broker and **Next.js** for the front-end client. **Lucia** and **JWT** handle authentication and authorization, allowing users to access data relevant to their roles. The system also supports **WebSocket** connections for real-time data updates.

## Architecture

The following key components make up the system architecture:

### 1. **Sensor Data Ingestion Service**
   - This service receives data from sensors (edge devices) via webhooks.
   - The service acts as a producer by publishing the incoming sensor data to a **message broker** (RabbitMQ).
   - Ensures no data is lost by using a reliable message queue.

### 2. **Message Broker (RabbitMQ)**
   - The central message bus that allows different services to subscribe to and process sensor data.
   - Each service pulls the necessary data from the message broker for its processing tasks.
   - Facilitates decoupled communication between services.

### 3. **Processing Services**
   - Each microservice subscribes to the data channels in RabbitMQ and processes it for specific purposes:
     - **Sensor Data Ingestion Service**: Receives and publishes the sensor data.
     - **Alert & Notification Service**: Generates alerts and notifications based on predefined thresholds.
     - **Production Tracking & Pipeline Monitoring Service**: Monitors the production line and tracks pipeline health.
     - **Energy Monitoring Service**: Monitors energy consumption metrics.
     - **Task Scheduling Service**: Manages and schedules tasks based on sensor inputs.
     - **Defect Logging Service**: Logs and tracks defects in the manufacturing pipeline.
     - **Authentication & Authorization Service**: Validates user identities and authorizes access to specific data streams.

### 4. **API Gateway**
   - The API Gateway subscribes to different message broker channels, based on the user's permissions.
   - It aggregates the processed data and makes it available to authenticated users.
   - The gateway ensures that only authorized users have access to specific topics based on their roles.
   - This is done using **Lucia** (for authentication) and **JWT** (for authorization).

### 5. **Real-Time WebSocket Connection**
   - The system supports **real-time communication** between the client and the API Gateway using **WebSocket** channels.
   - Once authenticated, the client establishes a WebSocket connection to receive real-time updates.
   - The gateway pushes real-time data from the message broker to the clients based on the user's subscribed topics.
   - This ensures that users get real-time updates without needing to refresh the page.

   **WebSocket Flow:**
   - Client initiates WebSocket connection upon login.
   - API Gateway authenticates the WebSocket connection using **JWT**.
   - The client subscribes to authorized channels (based on their role).
   - API Gateway listens to the relevant RabbitMQ channels and pushes real-time updates through WebSocket to the client.

### 6. **Client-Side (Next.js)**
   - Authenticated users can access the dashboard via a **Next.js** frontend.
   - Based on their role, they can subscribe to certain topics and monitor relevant information in real-time.
   - Roles determine which channels users can subscribe to and what data they can view.
   - **WebSocket** is used for real-time data streaming, ensuring low-latency updates.

### 7. **User Roles and Access Control**
   - Authentication and authorization are handled using **Lucia** with **JWT** tokens.
   - Users are assigned roles that control what data streams they can subscribe to.
   - Data security is enforced to ensure users only access information relevant to them.
