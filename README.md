# AWS-WordPressWebApp
Overview: This project aims to design and implement a robust, multi-tier WordPress web application infrastructure on Amazon Web Services (AWS), incorporating Virtual Private Cloud (VPC), Elastic Compute Cloud (EC2), Application Load Balancer (ALB), Relational Database Service (RDS), and Elastic File System (EFS). The primary goal is to achieve high availability and fault tolerance while maintaining scalability and security. I refrenced AOS notes tutorial when building this architecture (@AOS Note is his youtube channel). 

Step 1: Building a Secure Network - Three Tier Application Infrastructure

"The multi-tier architecture pattern provides a general framework to ensure decoupled and independently scalable application components can be separately developed, managed, and maintained" (from the AWS website). 

![WebApplicationStage1](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/092605ba-29ba-4218-a83c-ee35cc594b05)

The first step in creating a word press web application is setting up the network. I started by creating a VPC in the us-east-1 region (figure 1). I then attached an internet gateway in order to grant internet access to my public subnets (figure 2). Next, I created the public and private subnets (figure 3). The public subnets act as the front facing web application that the user directly interacts with. The first and second private subnets are used for the app tier of the infrastructure and handles traffic from the application load balancer and will contain EC2 instances running the Wordpress installation. The third and fourth subnets act as a the database tier where data such as the wordpress files will be located. In order to give proper internet access to the public subnets and proper routing of the vpc to the private subnets route tables are required. I created a public route table that has an internet gateway attached to it to grant internet access to the public subnet and a private route table that does not (figure 4). These were subsequently attached to the public and private subnets respectively. 

Figure 1
![vpc](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/5606d210-5c04-4380-8572-d6021c8e1e8f)

Figure 2
![internetgateway](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/0513c804-2b9d-4aec-8aa3-59f3b442f4d9)

Figure 3
![Subnets](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/c9290c54-6b80-4cb9-8673-090ca91973c6)

Figure 4
![RoutTables](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/daa7390b-444c-4943-805e-f3ffda44d709)

Step 2: Attaching a NAT Gateway

Nat Gateways are used in order to give internet access to the application tier so that the EC2 instances can download patches while users are not able to access the EC2 instances. Keeping the EC2 instance in the private subnet is security best practice since it protects our EC2 instances from being accessed by users on the internet and potentially executing malicious attacks on the instances. 
