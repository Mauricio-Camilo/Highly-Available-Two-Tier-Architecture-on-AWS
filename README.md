# Highly Available Two-Tier Architecture on AWS

This project demonstrates the deployment of a highly available two-tier architecture on AWS for a monolithic Node.js application.

The architecture is based on a lab created by Lucy Wang (Tech With Lucy). Beyond implementing the solution, my main goal was to understand the reasoning behind each architectural decision and document the concepts learned throughout the project.

---

## Architecture

<p align="center">
  <img src="./images/architecture.png" width="900">
</p>

---

## Business Problem

The application initially runs as a monolithic Node.js application on a single virtual machine.

Although simple to deploy, this architecture presents several limitations:

- Single point of failure
- Application and database hosted on the same server
- No load balancing
- Downtime during maintenance
- Limited scalability

The objective of this project was to redesign the infrastructure to improve availability, security, and scalability while keeping the application itself unchanged.

---

## Solution Overview

The proposed architecture separates the application and database into different layers.

Incoming requests are distributed through an Application Load Balancer across two EC2 instances deployed in different Availability Zones. The application stores its data in an Amazon RDS MySQL database deployed inside private subnets.

The infrastructure consists of:

- Amazon VPC
- Public and Private Subnets
- Internet Gateway
- Route Tables
- Security Groups
- Application Load Balancer
- Two Amazon EC2 instances
- Amazon RDS for MySQL
- Bastion Host

---

## Implementation

### Networking

The project was an excellent opportunity to better understand how networking works inside an AWS Virtual Private Cloud.

One of the most important concepts I learned is that **subnets are not inherently public or private**.

Whether a subnet is public or private depends on its routing configuration.

#### Public Subnets

The public subnets are associated with a Route Table containing a route to an Internet Gateway.

```text
0.0.0.0/0 → Internet Gateway
```

This allows resources such as:

- Application Load Balancer
- EC2 instances

to communicate with the Internet.

#### Private Subnets

The private subnets use a different Route Table without any route pointing to an Internet Gateway.

Because of this, resources inside these subnets cannot be accessed directly from the Internet.

Amazon RDS is deployed exclusively inside these private subnets to reduce its exposure to external threats.

> 📷 **VPC overview (subnets, route tables, and Internet Gateway)**

---

### Security

Another important concept I learned was how Security Groups work together to control communication between different layers of the application.

#### Application Load Balancer

The ALB Security Group allows:

- HTTP (80) from anywhere

This enables the load balancer to receive incoming client requests.

#### EC2 Instances

The EC2 Security Group allows:

- SSH (22) for administrative access
- HTTP (80) only from the ALB Security Group

Instead of exposing the application directly to the Internet, only the Application Load Balancer can communicate with the application servers.

This significantly reduces the application's attack surface.

#### Amazon RDS

The RDS Security Group allows:

- MySQL (3306) only from the EC2 Security Group

This prevents any external client from accessing the database directly.

> 📷 **Security Groups configuration**

---

### Database

The database layer was deployed using Amazon RDS for MySQL.

Some important security decisions include:

- Database Subnet Group composed only of private subnets
- Public Access disabled
- Access restricted through Security Groups

Since the database is not publicly accessible, a Bastion Host was used for administrative tasks such as creating the database schema and connecting through MySQL Workbench.

> 📷 **Amazon RDS configuration**

---

### Compute Layer

The application runs on two Amazon EC2 instances using Amazon Linux.

The deployment process consisted of:

- Connecting through SSH using an EC2 Key Pair
- Cloning the application from GitHub
- Installing Node.js
- Installing PM2 to manage the application process
- Installing and configuring Nginx as a reverse proxy

The deployment was intentionally performed manually, as the objective of this project was to understand AWS architecture rather than deployment automation.

> 📷 **Application running on the EC2 instances**

---

### Load Balancer

Traffic is distributed using an Application Load Balancer.

Configuration includes:

- HTTP Listener (Port 80)
- Target Group containing both EC2 instances
- Health Check configured on the `/` endpoint

If one instance becomes unhealthy, traffic is automatically routed to the remaining healthy instance.

> 📷 **Application Load Balancer configuration**

---

## High Availability

Compared to the original architecture, this solution removes the application's single point of failure.

High availability is achieved through:

- Multiple EC2 instances
- Application Load Balancer
- Deployment across multiple Availability Zones
- Database isolation in private subnets

The database remains deployed as a Single-AZ RDS instance to reduce costs for this learning project.

In a production environment, enabling Amazon RDS Multi-AZ would provide automatic failover and improve database availability.

---

## What I Learned

Building this architecture helped me deepen my understanding of several AWS networking and infrastructure concepts:

- **VPC Networking:** Understanding how Route Tables and Internet Gateways determine whether a subnet is public or private.
- **Network Security:** Restricting communication between application layers using Security Groups (ALB → EC2 → RDS).
- **Database Isolation:** Deploying Amazon RDS in private subnets with Public Access disabled to minimize external exposure.
- **Application Deployment:** Hosting a Node.js application on Amazon Linux using Nginx as a reverse proxy and PM2 for process management.
- **High Availability:** Using an Application Load Balancer to distribute traffic across multiple EC2 instances, eliminating the application's single point of failure.

---

## Future Improvements

While this architecture already addresses the application's current requirements, supporting higher traffic and increased operational demands would justify further architectural improvements:

- Configuring an Auto Scaling Group to automatically adjust application capacity.
- Deploying Amazon RDS in Multi-AZ for higher database availability.
- Hosting the frontend with Amazon S3 and CloudFront to improve performance and reduce the load on the application servers.
- Migrating the application to Amazon ECS Fargate to simplify deployments and improve operational scalability.
- Automating deployments using Terraform and GitHub Actions.

---

## Acknowledgements

This project is based on the AWS lab created by **Lucy Wang (Tech With Lucy)**.

The implementation, documentation, architectural analysis, and technical explanations in this repository reflect my own understanding and learning throughout the project.
