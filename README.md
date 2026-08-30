# AWS Three-Tier Highly Available Web Application Architecture

## 📌 Project Overview

This project demonstrates the design and deployment of a **secure, scalable, and highly available three-tier web application architecture on AWS**.

The architecture separates the application into three layers:

1. **Web Tier** – Application Load Balancer (ALB)
2. **Application Tier** – Amazon EC2 instances
3. **Database Tier** – Amazon RDS

The infrastructure is deployed inside a custom **Amazon VPC** using public and private subnets. A **NAT Gateway** provides outbound internet access for resources in private subnets without exposing them directly to the internet.

---

## 🏗️ Architecture

```text
                         INTERNET
                            │
                            ▼
                  ┌──────────────────┐
                  │       ALB        │
                  │ Application      │
                  │ Load Balancer    │
                  └────────┬─────────┘
                           │
                 ┌─────────┴─────────┐
                 │                   │
                 ▼                   ▼
          ┌─────────────┐     ┌─────────────┐
          │    EC2-1    │     │    EC2-2    │
          │ App Server  │     │ App Server  │
          └──────┬──────┘     └──────┬──────┘
                 │                   │
                 └─────────┬─────────┘
                           ▼
                    ┌─────────────┐
                    │     RDS     │
                    │  Database   │
                    └─────────────┘


                         AWS VPC
        ┌─────────────────────────────────────┐
        │                                     │
        │  PUBLIC SUBNETS                     │
        │  ┌───────────────────────────────┐  │
        │  │ ALB                           │  │
        │  └───────────────────────────────┘  │
        │                                     │
        │  PRIVATE APP SUBNETS                │
        │  ┌─────────────┐ ┌─────────────┐   │
        │  │ EC2-1       │ │ EC2-2       │   │
        │  └─────────────┘ └─────────────┘   │
        │          │              │            │
        │          └──────┬───────┘            │
        │                 ▼                    │
        │          NAT GATEWAY                 │
        │                 │                    │
        │                 ▼                    │
        │             INTERNET                 │
        │                                     │
        │  PRIVATE DB SUBNETS                 │
        │  ┌───────────────────────────────┐  │
        │  │ RDS                           │  │
        │  │ Database                      │  │
        │  └───────────────────────────────┘  │
        │                                     │
        └─────────────────────────────────────┘
```

---

## 🛠️ AWS Services Used

| AWS Service                   | Purpose                                                |
| ----------------------------- | ------------------------------------------------------ |
| **Amazon VPC**                | Creates an isolated network environment                |
| **Subnets**                   | Separates public, application, and database resources  |
| **Internet Gateway**          | Provides internet connectivity for public resources    |
| **Application Load Balancer** | Distributes incoming traffic across EC2 instances      |
| **Amazon EC2**                | Hosts the application/web servers                      |
| **Amazon RDS**                | Provides managed relational database storage           |
| **NAT Gateway**               | Provides outbound internet access from private subnets |
| **Security Groups**           | Controls network traffic between tiers                 |
| **Route Tables**              | Controls network routing                               |

---

# 🎯 Project Objectives

The main objectives of this project are:

* Design a three-tier AWS architecture.
* Create a custom VPC.
* Create public and private subnets.
* Configure an Internet Gateway.
* Configure a NAT Gateway.
* Deploy multiple EC2 application servers.
* Configure an Application Load Balancer.
* Create a Target Group.
* Deploy an Amazon RDS database.
* Implement tier-based Security Groups.
* Test application connectivity.
* Demonstrate a secure and highly available architecture.

---

# 🌐 Three-Tier Architecture

## 1. Web Tier

The web tier contains the **Application Load Balancer**.

The ALB receives requests from users and distributes traffic to healthy EC2 instances.

```text
User
  ↓
ALB
  ↓
EC2-1 / EC2-2
```

### Benefits

* Distributes traffic.
* Performs health checks.
* Improves availability.
* Prevents direct access to application servers.

---

# 2. Application Tier

The application tier contains multiple **Amazon EC2 instances**.

For example:

```text
EC2-1
EC2-2
```

The EC2 instances run the web/application server.

The instances are placed in **private subnets** so they are not directly accessible from the public internet.

Traffic reaches them through the ALB.

```text
Internet
   ↓
ALB
   ↓
Private EC2
```

---

# 3. Database Tier

The database tier contains **Amazon RDS**.

The RDS database is placed in private database subnets.

Users cannot directly access the database from the internet.

The application servers communicate with RDS.

```text
EC2
 ↓
RDS
```

This improves security by keeping the database isolated from public access.

---

# 🔐 Security Group Design

Security Groups are configured based on the three-tier architecture.

### ALB Security Group

Allow:

```text
HTTP  - Port 80  - 0.0.0.0/0
HTTPS - Port 443 - 0.0.0.0/0
```

The ALB accepts web traffic from users.

---

### EC2 Security Group

Allow:

```text
HTTP - Port 80 - ALB Security Group
```

SSH can be allowed only from a trusted administrator IP when required.

The EC2 instances should not allow HTTP traffic from the entire internet.

---

### RDS Security Group

Allow the database port only from the EC2 Security Group.

For MySQL:

```text
MySQL - Port 3306 - EC2 Security Group
```

This creates controlled communication:

```text
Internet
   ↓
 ALB
   ↓
 EC2
   ↓
 RDS
```

---

# 🌐 Network Design

The VPC is divided into multiple subnets.

Example:

```text
VPC
│
├── Public Subnet 1
│   └── ALB
│
├── Public Subnet 2
│   └── ALB
│
├── Private App Subnet 1
│   └── EC2-1
│
├── Private App Subnet 2
│   └── EC2-2
│
├── Private DB Subnet 1
│   └── RDS
│
└── Private DB Subnet 2
    └── RDS
```

The resources are distributed across multiple Availability Zones to improve availability.

---

# 🚪 Internet Gateway

The Internet Gateway provides internet connectivity to resources in public subnets.

The public route table contains:

```text
Destination: 0.0.0.0/0
Target: Internet Gateway
```

The ALB can therefore receive traffic from the internet.

---

# 🔄 NAT Gateway

The NAT Gateway allows resources in private subnets to access the internet for outbound connections.

For example, an EC2 instance may need to:

* Download software packages
* Install security updates
* Access external services

Traffic flow:

```text
Private EC2
    ↓
NAT Gateway
    ↓
Internet Gateway
    ↓
Internet
```

The NAT Gateway does **not** make the private EC2 instance directly reachable from the internet.

---

# ⚖️ Application Load Balancer

The ALB distributes incoming requests between EC2 instances.

Example:

```text
              User
                │
                ▼
               ALB
             /     \
            /       \
           ▼         ▼
        EC2-1      EC2-2
```

If EC2-1 becomes unhealthy, the ALB can stop sending traffic to it and continue using healthy targets.

---

# ❤️ Health Checks

The Target Group performs health checks against the EC2 instances.

Example:

```text
Protocol: HTTP
Port: 80
Path: /
```

If the application responds successfully, the instance is considered healthy.

```text
ALB
 │
 ├── EC2-1 → Healthy ✅
 │
 └── EC2-2 → Healthy ✅
```

---

# 🗄️ Amazon RDS

Amazon RDS is used as the managed database layer.

Example database:

```text
Amazon RDS MySQL
```

The application connects to the database using the RDS endpoint.

```text
Application
     ↓
RDS Endpoint
     ↓
MySQL Database
```

The database should remain in private subnets and should not be publicly accessible.

---

# 🚀 Implementation Steps

## Step 1 — Create VPC

Create a custom VPC.

Example:

```text
VPC CIDR:
10.0.0.0/16
```

---

## Step 2 — Create Subnets

Create public and private subnets across multiple Availability Zones.

Example:

```text
Public Subnet 1
10.0.1.0/24

Public Subnet 2
10.0.2.0/24

Private App Subnet 1
10.0.3.0/24

Private App Subnet 2
10.0.4.0/24

Private DB Subnet 1
10.0.5.0/24

Private DB Subnet 2
10.0.6.0/24
```

---

## Step 3 — Create Internet Gateway

Create an Internet Gateway and attach it to the VPC.

---

## Step 4 — Configure Route Tables

### Public Route Table

```text
0.0.0.0/0 → Internet Gateway
```

Associate it with the public subnets.

### Private Application Route Table

```text
0.0.0.0/0 → NAT Gateway
```

Associate it with the private application subnets.

### Database Route Table

The database subnets do not need a direct internet route.

---

## Step 5 — Create NAT Gateway

Create the NAT Gateway in a public subnet.

Associate an Elastic IP with the NAT Gateway.

---

## Step 6 — Create Security Groups

Create separate Security Groups for:

```text
ALB
EC2
RDS
```

Allow only the required traffic between the tiers.

---

## Step 7 — Launch EC2 Instances

Launch two EC2 instances in private application subnets.

Install the web/application server.

Example:

```bash
sudo dnf install httpd -y
sudo systemctl enable httpd
sudo systemctl start httpd
```

Create a test web page:

```bash
echo "AWS Three-Tier Application Server" | sudo tee /var/www/html/index.html
```

---

## Step 8 — Create Target Group

Create an ALB Target Group.

Example:

```text
Target Type: Instances
Protocol: HTTP
Port: 80
Health Check Path: /
```

Register both EC2 instances.

---

## Step 9 — Create Application Load Balancer

Create an internet-facing ALB.

Select the public subnets.

Attach the ALB Security Group.

Create an HTTP listener:

```text
HTTP : 80
        ↓
Target Group
        ↓
EC2 Instances
```

---

## Step 10 — Create RDS Database

Create an Amazon RDS database.

Example:

```text
Engine: MySQL
Deployment: Multi-AZ if required
Public Access: No
```

Place RDS in the private database subnets.

Attach the RDS Security Group.

---

# 🧪 Testing

After deployment, copy the ALB DNS name.

Example:

```text
http://<ALB-DNS-NAME>
```

Open it in a browser.

Expected result:

```text
AWS Three-Tier Application Server
```

The request flow is:

```text
Browser
   ↓
Internet
   ↓
Application Load Balancer
   ↓
Target Group
   ↓
EC2 Application Server
   ↓
RDS Database
```

---

# 📸 Project Screenshots

Add screenshots to the `screenshots/` directory.

Recommended screenshots:

1. VPC configuration
2. Subnet configuration
3. Route tables
4. Internet Gateway
5. NAT Gateway
6. Security Groups
7. EC2 instances
8. Target Group
9. Application Load Balancer
10. RDS database
11. Working application

Example:

```text
screenshots/
├── vpc.png
├── subnets.png
├── route-tables.png
├── nat-gateway.png
├── security-groups.png
├── ec2.png
├── target-group.png
├── alb.png
├── rds.png
└── website.png
```

---

# 🔒 Security Considerations

The architecture follows basic AWS security best practices:

* Keep EC2 application servers in private subnets.
* Keep RDS in private database subnets.
* Do not expose RDS directly to the internet.
* Allow EC2 traffic from the ALB Security Group.
* Allow RDS traffic only from the EC2 Security Group.
* Use least-privilege network access.
* Restrict SSH access to trusted IP addresses.
* Use HTTPS in production.
* Enable monitoring and logging.

---

# 📈 High Availability

The application uses multiple EC2 instances and multiple Availability Zones.

Example:

```text
             ALB
            /   \
           /     \
       AZ-1       AZ-2
        │           │
      EC2-1       EC2-2
        │           │
        └─────┬─────┘
              │
             RDS
```

If one application server becomes unavailable, the ALB can route traffic to the remaining healthy instance.

---

# 💡 Key Learnings

Through this project, I learned:

* AWS VPC architecture
* Public and private subnet design
* Route tables
* Internet Gateway
* NAT Gateway
* Application Load Balancer
* Target Groups
* EC2 deployment
* Amazon RDS
* Security Group design
* Availability Zones
* Three-tier application architecture
* Basic AWS security best practices
* High-availability architecture

---

# 🧰 Technologies

```text
AWS
Amazon VPC
EC2
Application Load Balancer
Amazon RDS
NAT Gateway
Internet Gateway
Security Groups
Linux
Apache
HTML/CSS
```

---

# 📁 Repository Structure

```text
aws-three-tier-architecture/
│
├── README.md
│
├── architecture/
│   └── three-tier-architecture.png
│
├── screenshots/
│   ├── vpc.png
│   ├── subnets.png
│   ├── route-tables.png
│   ├── nat-gateway.png
│   ├── security-groups.png
│   ├── ec2.png
│   ├── target-group.png
│   ├── alb.png
│   ├── rds.png
│   └── website.png
│
└── docs/
    └── project-explanation.md
```

---

# 🎤 Interview Explanation

### Tell me about your AWS Three-Tier Architecture project.

> I designed and deployed a three-tier web application architecture on AWS using VPC, Application Load Balancer, EC2, RDS, and NAT Gateway.
>
> I created a custom VPC with public and private subnets across multiple Availability Zones. The Application Load Balancer was deployed in the public subnets and distributed incoming traffic to EC2 application servers running in private subnets.
>
> The database layer used Amazon RDS in private database subnets. I configured Security Groups so that users could access the ALB, the ALB could communicate with EC2, and only the EC2 instances could communicate with the RDS database.
>
> I also configured a NAT Gateway to provide outbound internet access to the private EC2 instances while keeping them inaccessible directly from the internet.
>
> This architecture improved security, availability, scalability, and separation of application components.

---

# 🔑 AWS Keywords

```text
AWS
Cloud Computing
Amazon VPC
VPC
Subnets
Public Subnet
Private Subnet
Route Tables
Internet Gateway
NAT Gateway
Application Load Balancer
ALB
Target Group
EC2
Amazon RDS
MySQL
Security Groups
Availability Zones
High Availability
Scalability
Fault Tolerance
Three-Tier Architecture
Network Security
Linux
Apache
Cloud Architecture
AWS Solutions Architect
```

---

# 👨‍💻 Author

**AWS Cloud / Solutions Architect Associate Project**

This project was created as a hands-on AWS portfolio project to demonstrate practical knowledge of cloud networking, compute, load balancing, database architecture, security, and high availability.
