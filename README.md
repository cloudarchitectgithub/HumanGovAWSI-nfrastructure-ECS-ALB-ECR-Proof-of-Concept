# HumanGov: Proof of Concept (POC) on AWS Elastic Container Service (ECS) fronted By Application Load Balancer (ALB) and Storing Docker Images on Elastic Container Registry (ECR)

<p align="center">
<img src="https://i.imgur.com/F4zBOe8.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

## Project Description
The "Human-Gov Infrastructure" project is an initiative aimed at creating and managing cloud infrastructure for a government-related application. The project involves setting up various AWS resources using Terraform, an infrastructure as code (IaC) tool. The primary objective is to deploy a scalable and reliable infrastructure that includes an Amazon DynamoDB table, an S3 bucket, and necessary security groups, ensuring secure and efficient data storage and retrieval.

## Project Objectives

The primary objective of the "Human-Gov Infrastructure" project is to design and deploy a secure, scalable, and reliable cloud infrastructure using Amazon Web Services (AWS) and Terraform. This infrastructure is intended to support a government-related application, ensuring efficient data storage, retrieval, and secure access. The project aims to leverage infrastructure as code (IaC) principles to automate the deployment process, enhance infrastructure manageability, and facilitate easy scaling and maintenance of the cloud resources. Key objectives include:

1. **Automate Infrastructure Deployment:**
   - Use Terraform to automate the creation and management of AWS resources, ensuring consistent and repeatable infrastructure setups.

2. **Ensure Security and Compliance:**
   - Implement security best practices, including configuring appropriate security groups and IAM policies, to safeguard the infrastructure and data.

3. **Enhance Scalability and Reliability:**
   - Design the infrastructure to be highly scalable and reliable, accommodating varying workloads and ensuring high availability.

4. **Optimize Cost Management:**
   - Utilize cost-effective AWS services and configurations, such as pay-per-request billing for DynamoDB, to manage and optimize cloud spending.

5. **Facilitate Easy Maintenance and Updates:**
   - Structure the Terraform configuration for easy maintenance, allowing for straightforward updates and modifications as the project evolves.

## Tools and Technologies

- **AWS Elastic Container Service (ECS):** To run Docker containers and manage application services.
- **AWS Fargate:** For serverless compute that runs containers.
- **AWS Elastic Container Registry (ECR):** To store Docker images.
- **Amazon Application Load Balancer (ALB):** For traffic distribution across multiple container instances.
- **Amazon CloudWatch:** For monitoring services and application performance.
- **Amazon DynamoDB:** To store employee data.
- **Amazon S3:** For static file storage.
- **Terraform:** To automate the provisioning of infrastructure.

## Project Solution

### Part 1: Setting up Terraform Configuration

1. **Terraform .tf file Configuration:**
   - Main Configuration (main.tf): This file defines the root module and specifies the AWS provider and the state-specific infrastructure module.
   - Output Configuration (outputs.tf): Outputs were configured to expose the necessary attributes of the created resources, such as the DynamoDB table name and the S3 bucket name.

2. **Module Definition:**
   - A module (aws_humangov_infrastructure) was created to encapsulate the resource definitions. This module includes the creation of a DynamoDB table, an S3 bucket, and a security group.
   - Outputs within the Module: The module's outputs.tf file was configured to output the relevant attributes of the resources, ensuring that these values can be referenced in the root module.

3. **AWS Resources:**
   - DynamoDB Table: Configured with pay-per-request billing, an "id" hash key, and appropriate tags.
   - S3 Bucket: Configured with necessary properties, tags, and region-specific settings.
   - Security Group: Set up to allow traffic on specific ports (e.g., 22 for SSH, 80 for HTTP) to ensure secure access to instances.

### Part 2: Setting up ECS Cluster and Task Definition

1. **ECS Cluster Creation:**
   - An ECS cluster named human-gov-cluster was created using AWS Fargate as the launch type. This serverless option allowed the application to run containers without provisioning EC2 instances, reducing management overhead.

2. **Containerized Application:**
   - The HumanGov application was containerized into two images:
     - A Python container containing the core business logic for employee management.
     - An Nginx container to act as a reverse proxy, managing HTTP requests and forwarding them to the Python application.
   - These Docker images were stored in Amazon ECR (Elastic Container Registry).

3. **Task Definition:**
   - A new ECS task definition was created for the HumanGov application. The task definition contained:
     - Two containers: one for the Python application and one for Nginx.
     - Resources: Each task was allocated 1 vCPU and 3 GB of memory.
     - Task Role: Assigned human-gov-ECS-execution-role to allow interaction with DynamoDB and S3 services.
     - Both containers were configured with the necessary environment variables, including AWS region, DynamoDB table name, and S3 bucket name.

### Part 3: Service Deployment and Load Balancer Integration

1. **Service Creation:**
   - A new ECS service was created for the human-gov application using AWS Fargate as the launch type. The service was configured to maintain two running tasks for high availability across two different availability zones.

2. **Application Load Balancer (ALB):**
   - An Application Load Balancer (ALB) was created to distribute traffic across the two ECS tasks. The ALB forwarded incoming HTTP traffic on port 80 to the Nginx container, which then routed the traffic to the Python container.
   - The target group was configured to ensure the ALB correctly forwarded traffic to the Nginx container, and the ALB's health check monitored the status of the Python application.

3. **Networking Configuration:**
   - The ECS service used the default VPC, and a security group was configured to allow traffic on port 80. The tasks ran in two different availability zones, ensuring high availability and resilience in case of a zone failure.

4. **Testing and Validation:**
   - After deployment, the application was tested by accessing it through the ALB's DNS name. The application was validated by performing actions such as adding new employee records, viewing employee data, and uploading documents.
   - Employee data was stored in DynamoDB, and documents were successfully uploaded to Amazon S3.

### Part 4: Monitoring, Scaling, and Cleanup

1. **CloudWatch Monitoring:**
   - Amazon CloudWatch was used to monitor the performance of the ECS tasks and ALB. Metrics such as CPU and memory utilization were tracked to ensure optimal performance.

2. **Auto-scaling:**
   - The ECS service was configured to auto-scale based on CPU utilization metrics. This ensured that as traffic increased, more tasks were launched to handle the load, and when traffic decreased, tasks were automatically terminated.

3. **Resource Cleanup:**
   - After testing and ensuring the application worked as expected, resources were cleaned up to avoid unnecessary costs:
     - The ECS tasks were stopped, and the service was deleted.
     - The ALB and its associated target groups were removed.
     - Docker images were cleaned up from Amazon ECR.
     - DynamoDB tables and S3 buckets were cleared of any residual data.

By achieving these objectives, the project aims to deliver a robust infrastructure that meets the operational requirements of the government-related application, ensuring efficient and secure service delivery.

## Step-by-Step Directions

### Part 1: Containerizing the HumanGov Application with Docker and Pushing to AWS ECR

#### Implementation Steps:

##### Step 1: IAM Role Creation:
- We created a new IAM role named "HumanGovECSExecutionRole" with the following specifications:
  - Trusted Entity Type: AWS Service
  - Service or Use Case: Elastic Container Service Task
  - Permissions:
    - AmazonS3FullAccess
    - AmazonDynamoDBFullAccess
    - AmazonECSTaskExecutionRolePolicy

<p align="center">
<img src="https://i.imgur.com/CTkg3I7.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

##### Step 2: Containerizing the Flask Python Application:
- Navigate to the Application Directory:
  - Changed to the application directory: cd human-gov-application/src in Terraform

<p align="center">
<img src="https://i.imgur.com/MCuaxM7.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

##### Step 3: Create Dockerfile for Flask App:
- Create Dockerfile in cloud9 human-gov-application/src/Dockerfile
- Utilized a Python 3.8-slim-buster base image.
- Set the working directory, copied requirements, installed dependencies, and copied the Flask application.
- Configured Gunicorn to run the Flask app.

<p align="center">
<img src="https://i.imgur.com/0aLkMpD.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

- Created a public ECR repository named "humangov-app" and pushed the Docker image.
- Push commands for humangov-app

<p align="center">
<img src="https://i.imgur.com/YLwGgET.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

##### Step 4: Containerizing the NGINX Application:
- Create NGINX Configuration Files:
  - Created a new directory for NGINX: mkdir nginx.

<p align="center">
<img src="https://i.imgur.com/CJTChgd.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

- Created nginx.conf to configure NGINX for reverse proxy.

<p align="center">
<img src="https://i.imgur.com/V52nScn.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

- Created proxy_params for additional proxy configurations.

<p align="center">
<img src="https://i.imgur.com/pf1lhJN.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

##### Step 5: Create Dockerfile for NGINX:
- Used NGINX alpine as a base image.
- Removed default NGINX configuration.
- Copied custom configuration files.
- Exposed port 80.
- Started NGINX.

<p align="center">
<img src="https://i.imgur.com/pQJerQz.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

##### Step 6: Create ECR Repository for NGINX:
- Create a new public ECR repository named "humangov-nginx."
- Push commands for humangov-nginx

<p align="center">
<img src="https://i.imgur.com/YLwGgET.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

- Pushed the NGINX Docker image to the repository.

### Part 2: AWS S3 Bucket and DynamoDB Table Provisioning with Terraform, and ECS Cluster Creation

Introduction: In the second part of this project, we focused on provisioning the necessary AWS resources using Terraform, specifically creating an S3 Bucket and a DynamoDB Table. Additionally, we manually created an ECS (Elastic Container Service) cluster to prepare for deploying the HumanGov application.

Key Considerations:
- AWS Fargate was chosen for the ECS cluster due to its requirement for more memory than the T2.micro EC2 instance, which is part of the AWS free tier.
- While AWS Fargate doesn't have a free tier, the incurred charges for this hands-on project are minimal, serving as an investment in my skills and career development.

##### Step 1: Disabling Managed Credentials and Exporting Access Key:
1. Disabled AWS managed credentials in Cloud9 settings.

<p align="center">
<img src="https://i.imgur.com/6t3bkS6.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

2. Created a new access key for the Cloud9 user in the AWS IAM console.

<p align="center">
<img src="https://i.imgur.com/pOyW3R4.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

3. Push keys to Terraform for authentication.

<p align="center">
<img src="https://i.imgur.com/2YxSAaP.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

##### Step 2: Terraform Configuration:
1. Checked Terraform configuration with terraform show.

<p align="center">
<img src="https://i.imgur.com/vcJcy2Q.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

2. Commented out EC2-related resource blocks in Terraform files to avoid creating unnecessary resources.
   - Modified modules/aws_humangov_infrastructure/main.tf to exclude EC2 security groups and instances.

<p align="center">
<img src="https://i.imgur.com/sBguGyJ.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

   - Modified modules/aws_humangov_infrastructure/output.tf and terraform/output.tf to exclude EC2-related outputs.

<p align="center">
<img src="https://i.imgur.com/ieu9DGk.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

3. Ensured that the variables.tf file contained only one state (e.g., "California").

<p align="center">
<img src="https://i.imgur.com/dYPON0C.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

##### Step 3: Provisioning AWS Resources with Terraform:
1. Navigated to the Terraform directory and executed terraform plan to preview changes.

<p align="center">
<img src="https://i.imgur.com/EMuqEbZ.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

2. Verified that Terraform planned to create the DynamoDB table, S3 bucket, and random string suffix.

<p align="center">
<img src="https://i.imgur.com/baJuZE9.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

3. Applied the Terraform changes using terraform apply, confirming the creation of DynamoDB table and S3 bucket.

<p align="center">
<img src="https://i.imgur.com/Sgs46eM.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

##### Step 4: Manually Creating ECS Cluster:
1. Navigated to the AWS Management Console and accessed the ECS service.

<p align="center">
<img src="https://i.imgur.com/GfIop0z.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

2. Created a new ECS cluster named "humangov-cluster" with the Fargate infrastructure.

<p align="center">
<img src="https://i.imgur.com/Ug3LSmK.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

<p align="center">
<img src="https://i.imgur.com/qDVJ3nM.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

### Part 3: Deployment on ECS and Final Testing

In this final phase of the project, we create a new task definition on Amazon Elastic Container Service (ECS), deploy the application using a service, and perform thorough testing to ensure everything is functioning correctly.

##### Step 1: Creating the ECS Task Definition
1. Navigate to Task Definitions: In the ECS console, click on 'Task Definitions' on the left-hand side.

<p align="center">
<img src="https://i.imgur.com/iU6E3L7.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

2. Create New Task Definition: Click on 'Create new task definition' and follow the guided steps:
   - Task Name: Enter human.gov-fullstack.

<p align="center">
<img src="https://i.imgur.com/zOqQHCg.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

   - Launch Type: Select AWS Fargate.

<p align="center">
<img src="https://i.imgur.com/HvaZ9bm.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

   - Operating System: Use Linux.

<p align="center">
<img src="https://i.imgur.com/e965BRt.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

   - Task Size: Set to 1 vCPU and 3 GB of memory.

<p align="center">
<img src="https://i.imgur.com/aUHD6nq.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

   - Task Role: Choose the human.gov ECS execution role.

<p align="center">
<img src="https://i.imgur.com/9oFGw4i.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

   - Task Execution Role: Select the same human.gov ECS execution role.

3. Configure Containers:
   - Python Application Container:
     - Name: app
     - Image URI: Get the URI from the ECR repository for human.gov app.
     - Essential Container: Yes
     - Container Port: 8000

   - Environment Variables: Add the following variables with their respective values:
     - AWS_BUCKET: [Your S3 Bucket Name]
     - AWS_DYNAMODB_TABLE: [Your DynamoDB Table Name]
     - AWS_REGION: us-west-1
     - US_STAGE: California

   - Nginx Container:
     - Name: nginx
     - Image URI: Get the URI from the ECR repository for nginx.
     - Essential Container: No
     - Container Port: 80

4. Logging: Enable logging for both containers using the default ECS log group.
5. Review and Create: After ensuring all configurations are correct, create the task definition.

##### Step 2: Creating the ECS Service
1. Navigate to Services: In the ECS console, click on 'Services'.

2. Create Service:
   - Cluster: Select human-gov.
   - Launch Type: Choose Fargate.
   - Service Name: Enter human-gov.
   - Desired Tasks: Set to 2 for high availability.

3. Networking:
   - VPC: Use the default VPC.
   - Subnets: Use existing subnets.
   - Security Group: Ensure the security group has port 80 open.

4. Load Balancer Configuration:
   - Type: Application Load Balancer (ALB)
   - Name: human-gov-LB
   - Container to Load Balance: Select nginx container.
   - Listener Port: 80
   - Target Group: Create a new target group named human-gov-TG.

5. Review and Create: After verifying all settings, create the service.

<p align="center">
<img src="https://i.imgur.com/7TB9yp2.jpeg" height="80%" width="80%" alt="Pic"/>
<br />
<br />

<p align="center">
<img src="https://i.imgur.com/nfXDeNH.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

##### Step 3: Testing the Application
1. Check Service Status: Ensure the service is active and both tasks are running. Verify that each task is distributed across different availability zones for high availability.

<p align="center">
<img src="https://i.imgur.com/VAWbR5K.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

2. Access the Application:
   - Obtain the DNS name of the ALB from the ECS service networking section.
   - Open the application in a browser using the DNS name.

<p align="center">
<img src="https://i.imgur.com/oeVlJQm.jpeg" height="80%" width="80%" alt="Pic"/>
<br />
<br />

3. Application Functionality:
   - View Employees: Initially, no employees should be listed.
   - Add New Employee: Test adding a new employee named Caron Elisabeth with the role of IT Manager and a sample ID.

<p align="center">
<img src="https://i.imgur.com/Tauo4tz.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

   - Verify Data: Check that the employee data is correctly saved in DynamoDB and the ID is stored in S3.

<p align="center">
<img src="https://i.imgur.com/iNqYGmP.png" height="80%" width="80%" alt="Pic"/>
<br />
<br />

## Project Conclusion

The HumanGov ALB + ECS + ECR implementation demonstrated a scalable and serverless solution for containerized applications on AWS. By leveraging Fargate for ECS, the project achieved a fully-managed, hands-off infrastructure, ensuring high availability through multiple availability zones. The integration of an Application Load Balancer and efficient use of ECR streamlined the deployment process. Employee data was successfully stored and retrieved using DynamoDB and S3, with real-time testing confirming that the application worked as intended.

### Challenges Encountered

1. **ECS Task Role Permissions:** Initially, the task role did not have the correct permissions to interact with DynamoDB and S3, causing the application to fail. Adjustments were made to grant the necessary permissions.
2. **Container Environment Variables:** Some environment variables for connecting to the AWS services were misconfigured, leading to connection errors during early testing.
3. **Load Balancer Configuration:** Misalignment between the ALB configuration and the Nginx container caused issues with routing traffic. Correcting the target group to point to the Nginx container resolved this.

### Lessons Learned

- **Task Definition Precision:** Ensuring precise task definition configurations is critical, especially for container-to-service communication and memory management.
- **Load Balancer Integration:** Proper configuration of ALB with containers is essential for traffic management in a microservice environment.
- **Automating with Terraform:** Infrastructure-as-code simplified the setup process, making it easier to replicate environments and reduce manual configuration errors.

### Future Improvements

- **CI/CD Pipeline:** Automating the Docker build and push process using a continuous integration/continuous deployment pipeline with AWS CodePipeline or Jenkins could streamline updates.
- **Secrets Management:** Moving sensitive configuration data, such as environment variables, to AWS Secrets Manager for improved security.
- **Monitoring and Alerting:** Expanding monitoring with Amazon CloudWatch to include alerts for service degradation or downtime, ensuring real-time notifications for troubleshooting.
