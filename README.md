🚌 University Transportation System - Cloud Architecture 

📖 Project Overview 

A highly scalable, real-time university transportation tracking and management system. Designed for cloud-native deployment on AWS, this system provides live vehicle telemetry, route management, and secure authentication for students, drivers, and dispatchers.  

🏗️ High-Level Cloud Architecture 

![Design Screenshot](architecture.png)

The infrastructure is designed for High Availability (HA) and robust security, utilizing a Multi-AZ deployment model across public and private subnets. 


Traffic Flow & Component Layers 

Clients (Web + Mobile): Connect via custom domain. 

Route 53 (DNS): Resolves domains and routes traffic to the load balancer. 

Application Load Balancer (ALB): Resides in Public Subnets (AZ-a, AZ-b). Handles SSL termination and intelligently routes /api traffic to the REST service and /ws traffic to the WebSocket service. 

ECS Cluster (Compute Layer): Resides in Private App Subnets (Multi-AZ). Runs containerized backend services: 

REST API Service: Handles CRUD operations (Routes, Schedules, Users). 

WebSocket Service: Manages real-time bidirectional communication for live GPS tracking. 

Amazon ElastiCache - Redis (Cache Layer): Resides in Private Cache Subnets (Multi-AZ). Acts as an in-memory data store for ephemeral location data and a Pub/Sub message broker for WebSocket horizontal scaling. 

Amazon RDS (Data Layer): Resides in Private DB Subnets (Multi-AZ). Serves as the primary relational database for persistent state. 

Amazon Cognito (Auth): Handles secure user registration, login, and JWT token issuance. 

💻 Tech Stack 

Frontend: Angular (Web Dashboard for Dispatchers), Flutter (Mobile Client for iOS & Android) 

Backend: ASP.NET Core (REST API & WebSockets, containerized in ECS, auto-scalable per service) 

Database: Microsoft SQL Server (Amazon RDS, Multi-AZ, Private Subnets) 

Caching & Pub/Sub: Redis Cluster (Amazon ElastiCache, Multi-AZ Cluster Mode, Pub/Sub for WebSocket horizontal scaling) 

Identity Provider: Amazon Cognito 

Infrastructure: AWS (VPC with Public & Private Subnets, Route 53, Application Load Balancer, ECS Cluster on Fargate/EC2, NAT Gateway per AZ, Multi-AZ deployments for RDS & Redis) 

 

 

🔒 Security & Networking Strategy 

Security is implemented using a Defense in Depth approach: 

VPC Isolation: Compute (ECS), Cache (Redis), and Database (RDS) layers are deployed strictly in Private Subnets with absolutely no public IP addresses assigned. 

NAT Gateway: Outbound internet access for private subnets (e.g., pulling Docker images) is securely routed through one NAT Gateway per AZ. 

Strict Security Groups (SGs): Only the ALB is publicly exposed on port 443 (HTTPS). 

The ECS instances only accept traffic originating from the ALB SG. 

Redis and RDS only accept traffic originating from the ECS SG. 

Authentication: All API routes are protected; clients must pass valid Cognito JWTs in the Authorization header. 

🚀 Deployment (Upcoming) 

(Instructions on how to provision the infrastructure using IaC (e.g., AWS CDK/Terraform) and how to run the Docker containers locally will be added here). 

 