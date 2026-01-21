# AWS ECS Fargate Application Deployment

------------------------------------------------------

This project demonstrates how to deploy a containerized application on
 **AWS using ECS Fargate**, with Docker images stored in **Amazon ECR**
 and traffic managed through an **Application Load Balancer (ALB)**.  
The architecture follows AWS best practices using **private subnets**, 
**security groups**, and **managed scaling**.

---------------------------------------------------------


## 🏗️ Architecture Overview

- Users access the application via an **Application Load Balancer** in a public subnet
- The ALB forwards traffic to an **ECS Fargate service** running in private subnets
- Docker images are stored and pulled from **Amazon Elastic Container Registry (ECR)**
- Security Groups control inbound and outbound traffic
- The application scales automatically based on demand

----

🚀 Why This Project Matters
This project helped me gain hands-on experience with AWS container services, which are commonly used in real production environments.
It also strengthened my understanding of cloud networking, security, and scalable application deployment.

---

## 📐 Architecture Diagram

<Image1>


---

## 🔧 Tech Stack

- **[EC2 Instance](#1-connect-to-the-ec2-instance)**
- **[Docker]()**
- **[IAM ROLE]()**
- **[Amazon ECR]()**
- **[AWS ECS (Fargate)]()**
- **[Security Groups & Target Groups]()**
- **[Application Load Balancer (ALB)]()**

---

## 1. EC2 Setup & Docker Image Deployment

### 1️⃣ Launch EC2 Instance

- Launch an **Amazon Linux** EC2 instance
- Allow **HTTP (80)** and **SSH (22)** in the security group

<Ec2-launch config image>
<ec2 after launch>

---

### 2️⃣ Connect to the EC2 Instance



```bash
ssh -i key.pem ec2-user@<EC2_PUBLIC_IP>

````
### 3️⃣ Install Git and Docker

```bash
sudo yum install git docker -y
```

<git install iamge>

### 4️⃣ Enable and Start Docker

```bash
sudo systemctl start docker
```

```bash
sudo systemctl enable docker
```

# Add the EC2 user to the Docker group (to run Docker without sudo):

```bash
sudo usermod -aG docker ec2-user
```
<Docker setup image>

⚠️ Logout and login again for group changes to take effect.

### 5️⃣ Clone the Git Repository

```bash
git clone <REPOSITORY_URL>
cd <REPOSITORY_NAME>

```

6️⃣ Build Docker Image

```bash
docker build -t website:latest .

```

### 7️⃣ Run the Docker Container

```bash

docker run -d -p 80:80 website:latest

```
---
### 8️⃣ Access the Application

Open your browser and access the application using:

```bash

http://<EC2_PUBLIC_IP>:80

```

<Image web browser>

✅ Outcome

The application runs successfully inside a Docker container and is accessible through the EC2 public IP, confirming that the Docker image and container setup is working as expected.

---

## 2.  Push Docker Image to Amazon ECR

---

## ❓ What is Amazon ECR?

**Amazon ECR (Elastic Container Registry)** is an AWS service used to store and manage **Docker container images** securely inside the AWS environment.

In simple words:
- ECR works like **Docker Hub**, but it is **fully managed by AWS**
- ECS services pull Docker images directly from **ECR**
- Images stored in ECR are private and secured using **IAM permissions**

## Amazon ECR Setup (Container Registry)

 1. Go to Elastic Container Registry (ECR)
 2. Click Create repository
 3. Choose:
 4. Visibility: Private
 5. Repository name: *website-repo*
 6. Click Create repository

---

## ⚠️ Why IAM Role is Required?

I already had a Docker image built on my **EC2 instance**, but I was **not able to push it directly to Amazon ECR**.

**Reason:**  
The EC2 instance did not have the required **IAM Role** to communicate with the ECR service.

In AWS, services **do not talk to each other automatically**.  
Permissions are controlled using **IAM Roles**.

---

## 🔐 What is an IAM Role?

An **IAM Role** is a set of permissions that can be assigned to AWS services.

- Roles allow AWS services to access other AWS services securely
- No access keys are stored on the EC2 instance
- The role provides **temporary credentials**

📌 **Example:**  
An IAM Role attached to EC2 allows the instance to push Docker images to **Amazon ECR**.

---

## 🏗️ Architecture Flow
 
 <Image architecture of role>

---

## Steps to create IAM Role

 1. Open IAM Console
 2. Navigate to IAM → Roles
 3. Click Create role
 4. Select Trusted Entity(AWS service)
 5. EC2-ECR-FullAccess-Role
 6. Role Name (``Ec2-ECR-permission-demo``)
 6. Click Create role
## CREATE One More Role For ECS Task EXecution
follow same step 
 1. Select Trusted Entity(AWS SErvice)
 2. Select Elastic Container Service (ECS)
 3. In Permissions Policy (``AmazonECSTaskExecutionRolePolicy``)
 4. Role Name (ECSTaskExecutionRole)
 5. Click Next

## 🔗 Attach IAM Role to EC2 Instance

After creating the IAM role, the next step is to **attach the role to the running EC2 instance** so it can communicate with **Amazon ECR**.

---

### Steps to Attach IAM Role

1. Go to the **EC2 Dashboard**
2. Click on **Instances**
3. Select your **running EC2 instance**
4. Click on **Actions**
5. Navigate to **Security**
6. Click on **Modify IAM Role**
7. In the **IAM Role** dropdown, select the role you created earlier
8. Click on **Update IAM Role**

---

<Modify role image>

---

## ✅ Result

- The IAM role is now attached to the EC2 instance
- EC2 can securely communicate with **Amazon ECR**
- Docker images can now be pushed to ECR without using access keys
- This follows **AWS security best practices**

---

## 🎯 Key Point

> Attaching an IAM role to EC2 is the **recommended and secure way** to allow EC2 instances to access other AWS services like **ECR**, instead of using hardcoded credentials.

---

## 3. Push Docker Image to Amazon ECR

As mentioned earlier, **without an IAM role**, the EC2 instance was not able to push Docker images to Amazon ECR.  
Now that the **IAM role is attached**, the EC2 instance has permission to push images to the ECR repository.


## 🪜 Steps to Push Image to ECR

### 1️⃣ Open ECR Push Commands
1. Go to **Amazon ECR**
2. Select the **ECR repository** you created
3. Click on **View push commands**

AWS provides **four commands** that need to be executed one by one.

<ECr push command image>

---

### 2️⃣ Authenticate Docker to Amazon ECR

- Loggin

```bash
aws ecr get-login-password --region ap-south-1 | \
docker login --username AWS --password-stdin \
335357805095.dkr.ecr.ap-south-1.amazonaws.com
```

- Build the Docker image

```bash
docker build -t website:latest .

```
- Tag the Docker Image

```bash
docker tag website:latest \
335357805095.dkr.ecr.ap-south-1.amazonaws.com/<repository-name>:latest
```

- Push the Image to ECR

```bash
docker push \
335357805095.dkr.ecr.ap-south-1.amazonaws.com/<repository-name>:latest

```
<Image>

---

✅ Verify Image in ECR

Go back to the Amazon ECR console

Open your repository

Confirm that the Docker image is listed successfull

<Image>


---


---
## 3. ECS Setup

## ❓ What is Amazon ECS?

**Amazon ECS (Elastic Container Service)** is an AWS-managed **container orchestration service** used to run and manage containerized applications.

In simple terms:
- ECS runs Docker containers for you
- It automatically handles **scaling**, **availability**, and **deployment**
- You don’t need to manage servers when using **Fargate**

## 🪜 Create an ECS Cluster

### Steps to Create ECS Cluster

1. Go to the **Amazon ECS** service
2. Click on **Get started**
3. Click **Create cluster**
4. Enter a **Cluster name** ``Website-cluster``
5. Under **Infrastructure**, select **Fargate only**
6. Click **Create**

⏳ It takes a couple of minutes for the ECS cluster to be created.

<Image of created ECS-cluster>

---

## ❓ What is AWS Fargate?

**AWS Fargate** is a **serverless compute engine for containers**.

- You don’t manage EC2 instances
- No OS installation or maintenance
- No patching or capacity planning
- AWS manages everything in the background

> Even though servers exist behind the scenes, **AWS manages them on your behalf**, which is why Fargate is called *serverless*.

---

## 🧩 Create a Task Definition

---

## ❓ What is a Task?

A **Task** is a running instance of a container in ECS.

---

## ❓ What is a Task Definition?

A **Task Definition** is a blueprint that tells ECS:
- Which Docker image to use
- CPU and memory requirements
- Container ports
- IAM permissions

---

## 🪜 Steps to Create Task Definition

1. In the ECS left panel, click **Task definitions**
2. Click **Create new task definition**
3. Configure the task:
   - **Task definition name:** ``Website-task-defination``
   - **Launch type:** AWS Fargate
   - **CPU:** 0.25 vCPU
   - **Memory:** 0.5 GB
   - **Task role:** None
   - **Task execution role:** Select **``ecsTaskExecutionRole``** 

4. Add container details:
   - **Container name:** website
   - **Image URI:**  
     - Go to **Amazon ECR**
     - Copy the **Image URI**
     - Paste it here
     - rest leave default
     - click on create

<Image task defination>
---

## ✅ Result

- ECS cluster is created
- Task definition is configured using the ECR image
- The application is now ready to be deployed using **ECS Fargate**

---

## 4. Deployment Configuration

---

### Steps to Run ECS Task

1. Go to **Amazon ECS**
2. Open your **Cluster**
3. Click on **Run Task**
4. Configure the task:
   - **Task definition family:** ``Website-task-defination``
   - **Launch type:** Fargate
   - **Platform version:** LATEST
   - Leave all other options as **default**
5. Click **Create**

<Image of ECS task defination>

---


## 🌐 Access Application Using Public IP

After the task starts running:

1. Go to **Cluster → Tasks**
2. Select the running task
3. Copy the **Public IP**
4. Paste the IP in a web browser


 <Task defination image>


❌ **Result:**  
You will see a **timeout error**.

<image of time out error>

---

## ❓ Why Timeout Error Occurs?

The timeout happens because:
- The task is running in the **default VPC**
- The attached **security group does not allow HTTP (port 80) inbound traffic**
- So the browser cannot reach the application

---

## 🔓 Allow HTTP (Port 80) in Security Group

### Steps to Open Port 80

1. Go to the **running ECS task**
2. Open the **Networking** section
3. Click on the attached **Security Group**
4. Click **Edit inbound rules**
5. Add a new rule:
   - **Type:** HTTP
   - **Port:** 80
   - **Source:** Anywhere (0.0.0.0/0)
6. Click **Save rules**

<Task security group image>

---

## ✅ Verify Application Access

- Go back to the browser
- Refresh the **Public IP**

🎉 The application should now load successfully.

<Image of success>

---
## 🔄 Create ECS Service
---

## ❓ What is an ECS Service?

An **ECS Service**:
- Keeps the required number of tasks running
- Automatically restarts failed tasks
- Integrates with **Load Balancer**
- Supports auto scaling

---

### Steps to Create ECS Service

1. Go to **Amazon ECS**
2. Open your **Cluster**
3. Click on **Services**
4. Click **Create**
5. **Task definition family:** ``Website-task-defination``
6. **select service name:**
7. Configure:
   - **Launch type:** Fargate
   - **Platform version:** LATEST

---


## 5. Load Balancer Setup

Running tasks directly using public IP is **not recommended for production**.  
To handle traffic properly, we should use an **Application Load Balancer (ALB)**.

## 🎯 Create Target Group

Before creating the load balancer, we need a **Target Group** where ECS tasks will be registered.


### Steps to Create Target Group

1. Go to **EC2 Dashboard**
2. In the left panel, click on **Target Groups**
3. Click **Create target group**
4. Configure the target group:
   - **Target type:** IP
   - **Target group name:** `website-tg`
   - Leave all other settings as **default**
5. Click **Next → Next**
6. Click **Create target group**

---

## 🌐 Create Application Load Balancer (ALB)

### Steps to Create ALB

1. In the **EC2 Dashboard**, click on **Load Balancers**
2. Click **Create load balancer**
3. Select **Application Load Balancer (ALB)**
4. Click **Create**

---

### ALB Configuration

- **Name:** `website-alb`
- **Scheme:** Internet-facing
- **IP address type:** IPv4
- **VPC:** Default VPC
- **Availability Zones:** Select all available AZs
- Leave remaining settings as **default**

### Listener & Target Group

- **Listener:** HTTP (Port 80)
- **Default action:** Forward to existing target group
- **Target group:** `website-tg`

<ALB Image>

---
## ✅ Result

- Target Group is created
- Application Load Balancer is configured
- ALB is ready to receive internet traffic
- ECS tasks will be registered using **IP-based targeting**

---

## 🔄 Create ECS Service with Load Balancer

After creating the **Application Load Balancer** and **Target Group**, the next step is to create an **ECS Service** and attach the load balancer to it.

---

### Steps to Create ECS Service

1. Go to **Amazon ECS**
2. Open your **Cluster**
3. Click on **Services**
4. Click **Create**

---

### Service Configuration

- **Launch type:** Fargate
- **Platform version:** LATEST
- **Service type:** Replica
- **Desired tasks:** 2
- **Select Deployment Option:** Rolling Update

## ❓ What is Rolling Update?

A **Rolling Update** is a deployment strategy where **new tasks are deployed gradually**, while the old tasks are stopped step by step.

This ensures:

- **Zero or minimal downtime**
- Application remains available during deployment
- Traffic is smoothly shifted to new tasks


This ensures that **two tasks** are always running for high availability.

---
### Load Balancer Configuration

Scroll down to the **Load Balancing** section and configure:
Mark on use Load Balancer
- **Load balancer type:** Application Load Balancer
- **Load balancer:** Use an existing load balancer
- **Select ALB:** `website-alb`
- **Use an existing listener**
- **Target group:** Use an existing target group
- **Target group name:** `website-tg`

<Image of ecs ALB config>

### Autoscaling Configuration

Scroll down to the **Auto Scaling** section and configure:
Mark on Auto Scaling 
- **Minimum Num of task: '2' Maximum Num of tasks '4'**
- **policy name:** website-policy-ecs
- **ECS service matrics** ECSServiceAverageCPUUtilization
- **target value:** 10
- **rest leave default**

scroll Down and Click on create

<Image service Created>

⏳ It takes a couple of minutes for the ECS cluster to be created.

---
After the ECS service is created successfully: Go to EC2 Dashboard Click on Load Balancers Copy the **DNS name (URL)** of the ALB.

<Image of alb creation>

Now Copy of that **DNS name (URL)** of the ALB Paste the URL inro Your web browser.
<final Image>

---

## 🏁 Project Completion