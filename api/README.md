# API Documentation

## Overview

The backend of this project is implemented as a REST API responsible for handling customer orders, delivery preferences, and order status tracking.

The API is deployed as a containerized service running on **Amazon ECS (Elastic Container Service)**. It acts as the core application layer of the system and communicates with the database and messaging services.

The API processes incoming requests from the web interface and performs the necessary operations such as storing orders, updating delivery information, and publishing order events.

---

## API Responsibilities

The backend API performs several core functions within the system:

- Accept customer orders
- Store order information in the database
- Record delivery preferences
- Track order status
- Publish order events to messaging systems

These operations simulate how a real e-commerce ordering system would operate.

---

## Core API Endpoints

### Create Order

**Endpoint**

```
POST /orders
```

**Description**

Creates a new customer order in the system.

**Example Request Body**

```json
{
"customerName": "John Doe",
"product": "Fresh Produce Package",
"quantity": 2,
"deliveryOption": "Delivery",
"deliveryWindow": "10AM - 12PM",
"specialInstructions": "Leave at front door"
}
```


**System Actions**

1. Order data is validated
2. Order is stored in the database
3. Order event is published to the messaging queue

---

### Get Order Status

**Endpoint**

```
GET /orders/{orderId}
```


**Description**

Returns the current status of a specific order.

**Example Response**

```json
{
"orderId": 1045,
"status": "Processing",
"deliveryWindow": "10AM - 12PM"
}
```


Possible status values:

- Received
- Processing
- Preparing
- Out for Delivery
- Delivered

---

### Update Delivery Preferences

**Endpoint**

```
PUT /orders/{orderId}/delivery
```


**Description**

Allows a customer to update delivery preferences such as delivery time or special instructions.

**Example Request Body**

```json
{
"deliveryWindow": "12PM - 2PM",
"specialInstructions": "Call on arrival"
}
```


---

## Event Publishing

After a new order is created, the API publishes an event to a messaging queue.

Example event flow:

```
Order Created
↓
API publishes event
↓
Message sent to Amazon SQS
↓
Inventory update simulation triggered
```

This demonstrates how modern systems use asynchronous communication between services.

---

## API Deployment

The API runs inside a container deployed using **Amazon ECS**.

Deployment components include:

- ECS Cluster
- ECS Service
- ECS Task Definition
- Container image running the API

This architecture allows the API to scale and run reliably in a cloud environment.

---

## Testing the API

The API was tested using **Postman** to verify that endpoints respond correctly and return expected results.

Tests included:

- Creating new orders
- Retrieving order status
- Updating delivery preferences
- Verifying event messages published to the queue

These tests confirmed that the backend services were functioning correctly.
