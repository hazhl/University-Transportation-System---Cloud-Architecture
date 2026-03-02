# Architecture Decision Records (ADRs)

This document tracks the major architectural and technical decisions made for the University Transportation System, including the context, the decision itself, and the resulting consequences.

---

## 1. Compute Layer: Amazon ECS vs. EC2 vs. Lambda

**Context:** The system requires running a REST API and a highly concurrent WebSocket service for live GPS tracking.

**Decision:** We chose **Amazon ECS (Elastic Container Service)** deployed across Multi-AZ private subnets behind an Application Load Balancer.

**Alternatives Considered:**

* **Raw EC2 instances:** Rejected because managing underlying OS patching, scaling groups, and manual deployments adds unnecessary operational overhead compared to container orchestration.
* **AWS Lambda + API Gateway:** Rejected because WebSockets require long-lived, persistent connections. Lambda has connection time limits and can incur high costs for continuously active real-time connections.

**Consequences:**

* **Pros:** Docker containerization ensures the local development environment perfectly matches production. ECS provides easier orchestration and scaling for long-lived WebSocket connections compared to Lambda (which has connection time limits) and requires less server maintenance than raw EC2 instances.
* **Cons:** Introduces a slight learning curve for managing Task Definitions and Elastic Container Registry (ECR).

## 2. Real-Time Messaging: Redis Pub/Sub for WebSockets

**Context:** Because the ECS cluster spans multiple Availability Zones, a student might be connected to Container A, while the bus they are tracking sends its GPS coordinates to Container B.
**Decision:** We implemented **Amazon ElastiCache (Redis)** as a message backplane for the WebSocket service. This architecture follows a distributed WebSocket backplane pattern.

**Consequences:**

* **Pros:** Allows horizontally scaled WebSocket servers to share state. When Container B receives a GPS update, it publishes it to Redis, which instantly broadcasts it to Container A, ensuring the student sees the live update.
* **Cons:** Adds an infrastructure dependency and slight cost, but it is mandatory for distributed real-time systems.

## 3. Database Selection: Relational (Amazon RDS) vs. NoSQL

**Context:** The system handles highly structured data with clear relationships (e.g., a Driver is assigned to a Vehicle, which follows a Route consisting of multiple Stops).
**Decision:** We selected a relational database (PostgreSQL / SQL Server) hosted on Amazon RDS in a Multi-AZ deployment.
Multi-AZ ensures automatic failover and high availability in case of primary database failure.

**Consequences:**

* **Pros:** Enforces strict data integrity and ACID compliance, which is crucial for scheduling and user assignments.
* **Cons:** Scaling writes horizontally is harder than with NoSQL, but our primary heavy workload is reading coordinates, which is mitigated by the caching layer.

## 4. Technology Stack: ASP.NET Core & Angular

**Context:** The backend requires high performance for concurrent WebSocket connections, and the frontend requires a complex, interactive dashboard for dispatchers.
**Decision:** We selected **ASP.NET Core** (with SignalR) for the backend services and **Angular** for the frontend web client.

**Consequences:**

* **Pros:** Both frameworks offer strong typing (C# and TypeScript), excellent enterprise-grade tooling, and structured dependency injection. SignalR integrates seamlessly with Redis for the WebSocket backplane.
* **Cons:** Steeper learning curve compared to lightweight scripting languages, but yields a more maintainable, production-ready codebase.

## 5. High Availability Strategy: Multi-AZ Deployment

**Context:** A university transportation system experiences massive traffic spikes during peak commute hours (morning and late afternoon). Any downtime during these windows disrupts thousands of students and drivers.
**Decision:** We designed a **Multi-Availability Zone (Multi-AZ)** architecture across all critical tiers (ALB, ECS Cluster, Redis, and RDS) in the `eu-central-1` region.

**Consequences:**

* **Pros:** Ensures complete fault tolerance. If one physical AWS data center (AZ) experiences an outage, the Application Load Balancer automatically routes traffic to the healthy containers in the second AZ, while RDS and ElastiCache automatically failover to their standbys. This also provides low-latency routing for users in the Middle East and Europe.
* **Cons:** Increases baseline infrastructure costs, as redundant compute and database instances are running continuously, and cross-AZ data transfer incurs slight networking fees.

## 6. Networking & Security Model: Defense in Depth (VPC Isolation)

**Context:** The system handles sensitive user data and real-time GPS coordinates. Exposing the database or compute layers directly to the internet is a critical security vulnerability.
**Decision:** We implemented a **Strict VPC Network Isolation** strategy using Public and Private Subnets, combined with **Amazon Cognito** for Identity and Access Management (IAM).

**Consequences:**

* **Pros:**
  * **Network Isolation:** Only the Application Load Balancer (ALB) and NAT Gateways reside in the Public Subnets. The ECS containers, Redis cluster, and RDS instances are entirely hidden in Private Subnets with no public IP addresses.
  * **Traffic Control:** Security Groups act as strict firewalls. The database only trusts the ECS containers, and the ECS containers only trust the ALB.
  * **Stateless Auth:** Cognito offloads the risk of storing user passwords and handles secure JWT token issuance.
* **Cons:** Managing private resources makes debugging slightly more complex. Developers cannot connect directly to the RDS database from their local machines without setting up a Bastion Host, VPN, or AWS Systems Manager (SSM) tunnel. NAT Gateways also introduce a fixed hourly networking cost.
