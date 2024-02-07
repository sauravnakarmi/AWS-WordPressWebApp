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

![NatGatewayDiagram](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/93b0a368-5f3b-4203-9f32-2fdb7a4d2363)

Nat Gateways are used in order to give internet access to the application tier so that the EC2 instances can download patches while users are not able to access the EC2 instances. Keeping the EC2 instance in the private subnet is security best practice since it protects our EC2 instances from being accessed by users on the internet and potentially executing malicious attacks on the instances. I started by creating 2 Elastic Ip addresses that would be used for the NAT Gateways (figure 5). I then created the NAT gateways and attached elastic ip addresses (figure 6). In order to properly route traffic from the EC2 instance to the NAT Gateway I had to add a route in the route table from the private route tables to the NAT Gateways (figure 7). 

Figure 5
![ElasticIp](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/3ba2c18d-ee70-4c91-987d-a422c2e85d87)

Figure 6
![NatGateway](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/921aa575-fcbe-4483-accb-42dd9405851a)

Figure 7
![NatGatewayRouteTable](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/d8c9a1a3-4267-4ded-bafc-5468d8280562)

Step 3: Controlling Traffic with Security Groups

There are 4 main security groups that must be made in order to ensure proper routing of traffic in the Web application (Figure 8). The first security group routes traffic from the internet into the application load balancer. The second security group only allows traffic from the application load balancer to reach the web servers. The third security group ensures that database servers only accepts traffic from the webservers. Finally, the EFS will only accept traffic from the web servers. 

Figure 8
![image](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/b5183c15-aa12-4a96-82e0-f64fb7561d3f)

Step 4: Creating a MYSQL Database

![image](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/1d56f4c1-bbb2-4742-9611-372bc7f05b58)

Putting a database in a private subnet enhances its security posture, reduces the risk of unauthorized access and data breaches, and helps organizations comply with regulatory requirements. In order to do so a database subnet group needs to be created so that our database can be added when we eventually create it (figure 9). We then create our database and add it to the database subnet group (Figure 10). 

Figure 9
![dbsubnets](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/74915534-cc31-49a4-84c4-e2ff35521662)

Figure 10
![image](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/8113a7c3-b36a-495f-ac89-bde71dab713e)

Step 5: Attaching an EFS

The primary purpose of Amazon EFS is to provide scalable, elastic, and highly available file storage.

![efsdiagram](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/38ec656d-7f4a-4d3d-aa0e-75d4d2540329)

In our case we are using the EFS as a way to store our application code, namely the WordPress installation, that our webservers will pull from. 

![efs](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/18477997-ef5f-4322-919a-47930cfc1568)

