# Microservices Architecture – Leave Management System

## Introduction

In simple words, microservices means dividing one large application into multiple small, independent applications. Each application is responsible for a particular business task.


<img width="1376" height="768" alt="Gemini_Generated_Image_gjibtngjibtngjib" src="https://github.com/user-attachments/assets/5aa10206-2e40-4ff0-84c4-6b444b158fa8" />


We will understand this architecture using a **Leave Management System**.

Imagine a college or company where:

- Employees have their own information.
- Employees can apply for leave.
- Administrators can approve or reject leave requests.
- The system maintains different leave types, such as Casual Leave and Sick Leave.

Instead of building all these features inside one large project, we divide them into three microservices.

## Microservices

### Employee Service

**Responsibility:** Manage employee information.

Example operations:

- Add Employee
- Update Employee
- Get Employee
- Get All Employees
- Delete Employee

**Database:** Employee Database (SQL Server)

This service owns and manages employee-related data.

### Leave Request Service

**Responsibility:** Manage employee leave applications.

Example operations:

- Apply for Leave
- Get Leave Requests
- Approve Leave
- Reject Leave
- Get Leave Request Details

**Database:** Leave Request Database (SQL Server)

This service owns leave-request-related data. When it needs employee information, it communicates with the Employee Service instead of directly accessing the Employee Database.

### Leave Type Service

**Responsibility:** Manage the types of leave available to employees.

Example operations:

- Add Leave Type
- Update Leave Type
- Get Leave Types
- Delete Leave Type

**Database:** Leave Type Database (SQL Server)

Examples of leave types:

- Casual Leave
- Sick Leave
- Paid Leave

This service owns leave-type-related data.

## Architecture Components

### Client Layer

The client layer is where users or other applications send requests.

Examples:

- Web Application – React or Angular
- Mobile Application
- API Client – Postman

For example, an employee clicks the **Apply Leave** button in the web application. The application sends a request to the backend.

### API Gateway

The API Gateway acts as the **single entry point** for client requests.

Instead of the client calling each microservice directly, it sends requests to the API Gateway. The gateway forwards each request to the correct service.

Main responsibilities:

- **Request Routing:** Sends requests to the correct microservice.
- **Authentication:** Checks the user's identity.
- **Authorization:** Checks whether the user has permission.
- **Load Balancing:** Distributes requests across available service instances, when configured.

Example:

```text
Client
  |
  v
API Gateway
  |
  +----> Employee Service
  |
  +----> Leave Request Service
  |
  +----> Leave Type Service
```

### Independent Databases

Each microservice has its own database.

| Microservice          | Database               |
| --------------------- | ---------------------- |
| Employee Service      | Employee Database      |
| Leave Request Service | Leave Request Database |
| Leave Type Service    | Leave Type Database    |

**Important:** One service should not directly access another service's database. If it needs another service's information, it should communicate through that service's API or an appropriate messaging mechanism.

This approach is called **Database per Service**.

## Communication Between Microservices

Microservices need to communicate when one service requires information or an action from another service.

There are two common communication patterns.

### Synchronous Communication

In synchronous communication, one service sends a request to another service and waits for its response.

Technologies:

- HTTP / REST API
- gRPC

```text
Leave Request Service
        |
        | Request: Get Employee Details
        v
   Employee Service
        |
        | Response: Employee Details
        v
Leave Request Service
```

### Asynchronous Communication

In asynchronous communication, a service sends a message to a message broker without waiting for another service to process it immediately.

Technology:

- RabbitMQ
- Apache Kafka

A **Message Broker** receives and stores messages until the appropriate consumer processes them.

```text
Leave Request Service
        |
        | Publish message
        v
     RabbitMQ
        |
        | Consume message
        v
 Notification Consumer
        |
        v
 Send Notification
```

## Synchronous vs Asynchronous Communication

| Point               | Synchronous                             | Asynchronous                             |
| ------------------- | --------------------------------------- | ---------------------------------------- |
| Meaning             | Request and wait for response           | Send a message and continue              |
| Waiting             | Sender waits                            | Sender does not wait for processing      |
| Common technologies | REST API, gRPC                          | RabbitMQ, Kafka                          |
| Dependency          | Receiver should respond during the call | Consumer can process the message later   |
| Example             | Get employee details                    | Send leave-request notification          |
| Main benefit        | Immediate response                      | Loose coupling and background processing |
