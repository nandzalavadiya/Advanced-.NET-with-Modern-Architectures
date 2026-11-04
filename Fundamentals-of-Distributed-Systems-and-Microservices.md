# Fundamentals of Distributed Systems & Microservices


[Uploading Fundamentals-of-Distributed-Systems-and-Microservices.md…]()

<img width="884" height="330" alt="MicroServices vs Distributed System" src="https://github.com/user-attachments/assets/703c7fc5-6f65-4f77-a342-a3a6a276da67" />



## 1. What is a Distributed System?

A **Distributed System** is a network of multiple independent computers (servers) that communicate and work together to provide services to users as a single, unified system.

In simple words, a Distributed System means **storing and processing data across multiple servers located in different cities or countries, where all servers communicate and work together.**

### Real-World Example: Netflix

Imagine Netflix has multiple data centers deployed in different locations:

- Mumbai Data Center (India)
- Bengaluru Data Center (India)
- Singapore Data Center (Singapore)

All these data centers work together to provide Netflix services to users.

**How does it work?**

1. Netflix stores and manages its data across multiple servers.
2. Users from different locations can access the same video content.
3. If a user watches a video in Mumbai, the system can synchronize their watch progress with other servers.
4. If a user likes or comments on a video, that information can be shared across the system.
5. Subscriber information and other important data are synchronized between different servers when required.

**Example:**

Suppose a user watches a movie on Netflix in Mumbai and stops at 25 minutes.

When the same user opens Netflix in Singapore, the system can show the movie's progress from 25 minutes.

Similarly, likes, comments, and subscriber-related information can be shared across the distributed infrastructure.

### Simple Diagram

```text
                  Netflix Application
                          |
              -------------------------
              |           |           |
              v           v           v
         Mumbai DC   Bengaluru DC  Singapore DC
              |           |           |
              v           v           v
          Server 1    Server 2    Server 3
              |           |           |
              -------- Data Sync -------
                          |
                  Shared Netflix Data
```

### Key Points

- Multiple independent computers work together.
- Servers can be located in different geographical locations.
- Servers communicate through a network.
- Data can be replicated across multiple servers.
- Users can access services from different locations.
- It improves scalability, availability, and fault tolerance.

## 2. What are Microservices?

**Microservices Architecture** is a software development approach in which a large application is divided into multiple small, independent services. Each service is responsible for a specific business functionality and communicates with other services through APIs or messaging.

In simple words, **Microservices means dividing one large application into multiple small applications (services), where each service handles its own specific task.**

### Real-World Example: Netflix

Imagine Netflix is a large application. Instead of developing the entire application as one single project, we can divide it into multiple independent services.

For example:

| Microservice           | Responsibility                         |
| ---------------------- | -------------------------------------- |
| User Service           | Register, Login, Manage Users          |
| Movie Service          | Manage Movies and Video Content        |
| Subscription Service   | Manage Subscription Plans              |
| Payment Service        | Handle Payments                        |
| Recommendation Service | Suggest Movies Based on User Interests |
| Review Service         | Manage Likes, Comments and Reviews     |
| Notification Service   | Send Email and App Notifications       |

**How does it work?**

1. A user opens the Netflix application and logs in.
2. The User Service verifies the user's login details.
3. The Subscription Service checks the user's subscription status.
4. The Movie Service provides available movies and video content.
5. The Recommendation Service suggests movies based on user interests.
6. If the user makes a payment, the Payment Service processes it.
7. The Notification Service sends payment or subscription notifications.

Each service performs its own responsibility and communicates with other services whenever required.

### Simple Diagram

```text
                   Netflix Application
                           |
                     API Gateway
                           |
       -----------------------------------------
       |          |          |         |       |
       v          v          v         v       v
    User      Movie/Video Subscription Payment Recommendation
   Service    Service     Service    Service    Service
       |          |          |         |       |
       v          v          v         v       v
    User DB    Movie DB   Subscription Payment Recommendation
                           DB          DB        DB


```

### Important Points

- Each microservice handles a specific business functionality.
- Every service can be developed and deployed independently.
- Each service can have its own database.
- Services communicate using REST APIs, gRPC, or messaging.
- Different services can use different programming languages.
- One service can be updated without redeploying the entire application.
- Each service can be scaled independently based on requirements.

## What is an API Gateway?

**API Gateway** is a single entry point between the Client (End User) and multiple Microservices. It receives client requests and automatically forwards them to the appropriate Microservice based on the requested API route.

### Example: Netflix

Suppose we have 3 Microservices running on different ports:

| Microservice         | Port |
| -------------------- | ---- |
| User Service         | 5101 |
| Subscription Service | 5102 |
| Payment Service      | 5103 |

**Without API Gateway:**

The client needs to call different URLs for different services.

- If a user wants to create an account, call `http://localhost:5101/` (User Service).
- If a user wants to manage a subscription, call `http://localhost:5102/` (Subscription Service).
- If a user wants to make a payment, call `http://localhost:5103/` (Payment Service).

This means the client needs to know the URL and port number of every Microservice.

**With API Gateway:**

Instead of calling different URLs and ports, the client only needs to call one common URL (API Gateway).

For example:

`http://localhost:5000/`

The API Gateway automatically redirects the request to the appropriate Microservice based on the requested API endpoint.

**Example:**

| Client Request    | API Gateway Redirects To    |
| ----------------- | --------------------------- |
| `/users/register` | User Service (5101)         |
| `/subscriptions`  | Subscription Service (5102) |
| `/payments`       | Payment Service (5103)      |

### Simple Diagram

```text
                  Client / End User
                          |
                          |
                          v
                  API Gateway
                 localhost:5000
                          |
          --------------------------------
          |               |              |
          v               v              v
      User Service  Subscription     Payment
        :5101        Service :5102   Service :5103
          |               |              |
          v               v              v
        User DB     Subscription DB   Payment DB
```
