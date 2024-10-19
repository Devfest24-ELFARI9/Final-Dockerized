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
This project implements a sensor data ingestion, processing, and monitoring system using a microservices architecture to support real-time production tracking, defect monitoring, and alerting in a manufacturing environment. The architecture is built to handle high-frequency sensor data, ensure scalability, and deliver low-latency updates to authenticated users via a web-based frontend.
Key technologies used include:

    -**RabbitMQ** as the central message broker for decoupled communication between services.
    -**Next.js** as the front-end client for real-time data visualization.
    -**Lucia** for authentication and JWT for authorization to control data access.
    -**WebSockets** for real-time updates to the client.
    -**InfluxDB** for time-series data storage with retention policies to manage storage efficiency.
    -**Express.js** for building the services apps.
    -**Prisma** for database management in services with SQL data models.
    -**MongoDB** for managing NoSQL data in services that handle non-relational datasets.
    -**Ngrok** to subscribe to the webhook provided while in local dev mode
**Note : each service has its own dedicated database.**

## Architecture

![image](https://github.com/user-attachments/assets/9e041c70-c8d7-4be7-b92d-32bb19efae8f)


The following key components make up the system architecture:

### 1. **Sensor Data Ingestion Service**
   - This service receives data from sensors (edge devices) via webhooks.
   - The service acts as a producer by publishing the incoming sensor data to a **message broker** (RabbitMQ).
   - It Pre-process the data and analyse it.
   - if a metric exceeds a certain threshold (indicating a problem) it published an alert message through the **message broker** so that it will be handled by **Alert & Notification Service**
   - Ensures no data is lost by using a reliable message queue.
   - It stores the sensors data in a time-series DB (influx) so that we can apply further analyses on the data in future
   - we apply retention policies on influx freeing up storage
   - **Technologies:** Nodejs , expressJs , InfluxDB , 

### 2. **Message Broker (RabbitMQ)**
   - The central message bus that allows different services to subscribe to and process sensor data.
   - Each service pulls the necessary data from the message broker for its processing tasks.
   - Facilitates decoupled communication between services.
   - ensures no data or messages loss
   - Each microservice subscribes to the data channels in RabbitMQ and processes it for specific purposes:
     - **Sensor Data Ingestion Service**: Receives and publishes the sensor data.
     - **Alert & Notification Service**: Generates alerts and notifications based on predefined thresholds.
     - **Production Tracking & Defect Monitoring Service**: Monitors the production line and tracks pipeline health and tracks defects in it.
     - **Task Scheduling Service**: Manages and schedules tasks based on sensor inputs.

### 4. **API Gateway**
   - The API Gateway subscribes to different message broker channels, based on the user's permissions.
   - It aggregates the processed data and makes it available to authenticated users by exposing endpoints and setting **websocket** connections with the front-end.
   - The gateway ensures that only authorized users have access to specific topics based on their roles.
   - This is done using **Lucia** (for authentication) and **JWT** (for authorization).
   - **Technologies:** Nodejs , expressJs , Prisma , PostgreSQL , Lucia , Websockets 


### 5. **Alert & Notification Service**
   - Receive alert triggers from other services (production monitoring service and sensor data ingestion service)
   - Notify the correct role (e.g., manager, operator) based on the issue (through the message broker => api gateway => user with the right role).
   - **Technologies:** Nodejs , expressJs , MongoDB (used to store alerts and notifications) 


### 6. **Task Scheduling Service**
   - receives events from other services mainly alerts and notifications service (subscribe to specific messages from message broker)
   - Generate maintenance tasks (e.g., schedule battery replacement).
   - Assign tasks to the right personnel based on roles.

### 7. **Production Tracking & defect Monitoring Service**
   - Monitor machine output and performance metrics (cars/day , avg time/cycle , revenu/day , defected cars/day , cars made/month) .
   - Compare actual production progress to targets.
   - Detect issues in the production line and notify appropriate personnel via the Alert Service.
   - **Note :** we used a Node.js script to simulate sending production data points to the monitoring service ,this script will generate random car arrival and departure events and send them to the /production-datapoint endpoint of the service
   - **Technologies:** Nodejs , expressJs , InfluxDB (to store the generated datapoints from machines) 


### 8. **Real-Time WebSocket Connection**
   - The system supports **real-time communication** between the client and the API Gateway using **WebSocket** channels.
   - Once authenticated, the client establishes a WebSocket connection to receive real-time updates.
   - The gateway pushes real-time data from the message broker to the clients based on the user's subscribed topics.
   - This ensures that users get real-time updates without needing to refresh the page.

   **WebSocket Flow:**
   - Client initiates WebSocket connection upon login.
   - API Gateway authenticates the WebSocket connection using **JWT**.
   - The client subscribes to authorized channels (based on their role).
   - API Gateway listens to the relevant RabbitMQ channels and pushes real-time updates through WebSocket to the client.

### 9. **Client-Side (Next.js)**
   - Authenticated users can access the dashboard via a **Next.js** frontend.
   - Based on their role, they can subscribe to certain topics and monitor relevant information in real-time.
   - Roles determine which channels users can subscribe to and what data they can view.
   - **WebSocket** is used for real-time data streaming, ensuring low-latency updates.

### 10. **User Roles and Access Control**
   - Authentication and authorization are handled using **Lucia** with **JWT** tokens.
   - Users are assigned roles that control what data streams they can subscribe to.
   - Data security is enforced to ensure users only access information relevant to them.
