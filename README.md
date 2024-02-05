# AWS-WordPressWebApp
Overview: This project aims to design and implement a robust, multi-tier WordPress web application infrastructure on Amazon Web Services (AWS), incorporating Virtual Private Cloud (VPC), Elastic Compute Cloud (EC2), Application Load Balancer (ALB), Relational Database Service (RDS), and Elastic File System (EFS). The primary goal is to achieve high availability and fault tolerance while maintaining scalability and security. I refrenced AOS notes tutorial when building this architecture (@AOS Note is his youtube channel). 

Step 1: Building a Secure Network - Three Tier Application Infrastructure
"The multi-tier architecture pattern provides a general framework to ensure decoupled and independently scalable application components can be separately developed, managed, and maintained" (from the AWS website). 

![WebApplicationStage1](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/092605ba-29ba-4218-a83c-ee35cc594b05)

The first step in creating a word press web application is setting up the network. I started by creating a VPC in the us-east-1 region. I then attached an internet gateway in order to grant internet access to my public subnets. Next, I created the public and private subnets. The public subnets act as the front facing web application that the user directly interacts with. The first and second private subnets are used for the app tier of the infrastructure and handles traffic from the application load balancer and will contain EC2 instances running the Wordpress installation. The third and fourth subnets act as a the database tier where data such as the wordpress files will be located. In order to give proper internet access to the public subnets and proper routing of the vpc to the private subnets route tables are required. I created a public route table that has an internet gateway attached to it to grant internet access to the public subnet and a private route table that does not have an internet gate way attached to it. These were subsequently attached to the public and private subnets respectively. 
