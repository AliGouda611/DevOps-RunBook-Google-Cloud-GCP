## 0. System Architecture Overview

This section provides an overview of the system architecture, detailing the various microservices and services involved in our infrastructure. Understanding the structure and purpose of these components will provide a solid foundation for grasping the subsequent sections of this documentation.

---

## Backend Microservices

Our system comprises four backend microservices, each catering to a different aspect of our operations. Here are the details of these microservices:

### 1. `backend`:
   This microservice serves as the core backend, handling the primary logic and data processing tasks required for our operations.

### 2. `backend-transportation`:
   Manages all transportation-related operations, such as booking and tracking transportation services.

### 3. `backend-hotels`:
   Manages hotel-related operations, such as searching for, booking, and managing hotel reservations.

### 4. `backend-payment`:
   Handles all payment transactions, including payment processing, payment verification, and refunds.

---

## Services

There are five additional services in our system architecture, each playing a crucial role in the overall functionality:

### 1. `ai`:
   Handles everything related to AI.

### 2. `frontend`:
   Serves as the user interface, enabling users to interact with our system.

### 3. `middleware`:
   Acts as a communication facilitator between the frontend and backend services, ensuring seamless data flow.

### 4. `third-party`:
   Manages interactions with third-party services and APIs.

### 5. `api-gateway`:
   Acts as a single entry point for client requests, directing them to the respective microservices and services.

---

In the following sections, we'll dive deeper into the configuration and setup of these components, elaborating on the operations and the challenges faced during the deployment and management processes.
