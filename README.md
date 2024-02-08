# AWS-WordPressWebApp
Overview: This project aims to design and implement a robust, multi-tier WordPress web application infrastructure on Amazon Web Services (AWS), incorporating Virtual Private Cloud (VPC), Elastic Compute Cloud (EC2), Application Load Balancer (ALB), Relational Database Service (RDS), and Elastic File System (EFS). The primary goal is to achieve high availability and fault tolerance while maintaining scalability and security. I refrenced AOS notes tutorial when building this architecture (@AOS Note is his youtube channel). 

## Step 1: Building a Secure Network - Three Tier Application Infrastructure

"The multi-tier architecture pattern provides a general framework to ensure decoupled and independently scalable application components can be separately developed, managed, and maintained" (from the AWS website). 

![WebApplicationStage1](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/092605ba-29ba-4218-a83c-ee35cc594b05)

The initial phase of crafting a WordPress web application involves network setup. Commencing with the creation of a VPC within the us-east-1 region, as illustrated in Figure 1, I proceeded by affixing an internet gateway to provide internet connectivity to the public subnets, depicted in Figure 2. Subsequently, I delineated both public and private subnets, as showcased in Figure 3. The public subnets serve as the interface for user interaction with the web application. Meanwhile, the first and second private subnets constitute the application tier, managing traffic from the application load balancer and housing EC2 instances housing the WordPress installation. The third and fourth subnets form the database tier, housing critical data such as WordPress files. To ensure proper internet access for the public subnets and efficient routing of VPC traffic to the private subnets, I crafted route tables. This included establishing a public route table, equipped with an internet gateway to furnish internet access to the public subnet, alongside a private route table devoid of such attachment, as depicted in Figure 4. These route tables were subsequently linked to their respective public and private subnets.

Figure 1
![vpc](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/5606d210-5c04-4380-8572-d6021c8e1e8f)

Figure 2
![internetgateway](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/0513c804-2b9d-4aec-8aa3-59f3b442f4d9)

Figure 3
![Subnets](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/c9290c54-6b80-4cb9-8673-090ca91973c6)

Figure 4
![RoutTables](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/daa7390b-444c-4943-805e-f3ffda44d709)

## Step 2: Attaching a NAT Gateway

![NatGatewayDiagram](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/93b0a368-5f3b-4203-9f32-2fdb7a4d2363)

NAT Gateways serve a crucial role in granting internet access to the application tier while safeguarding EC2 instances from direct user access, thereby adhering to security best practices. This setup allows EC2 instances to download patches seamlessly without user intervention. Initiating the process, I provisioned two Elastic IP addresses designated for NAT Gateways, as delineated in Figure 5. Subsequently, the NAT Gateways were instantiated and associated with their respective Elastic IP addresses, as depicted in Figure 6. To ensure smooth traffic routing from the EC2 instances to the NAT Gateways, I configured route tables within the private subnet, establishing routes directing traffic to the NAT Gateways, as outlined in Figure 7.

Figure 5
![ElasticIp](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/3ba2c18d-ee70-4c91-987d-a422c2e85d87)

Figure 6
![NatGateway](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/921aa575-fcbe-4483-accb-42dd9405851a)

Figure 7
![NatGatewayRouteTable](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/d8c9a1a3-4267-4ded-bafc-5468d8280562)

## Step 3: Controlling Traffic with Security Groups

To ensure seamless traffic routing within the web application (as depicted in Figure 8), four primary security groups are imperative. The initial security group facilitates the ingress of traffic from the internet into the application load balancer. Subsequently, the second security group meticulously restricts traffic exclusively from the application load balancer to access the web servers. Furthermore, the third security group meticulously governs that database servers solely accept traffic originating from the web servers. Lastly, the EFS is configured to solely accept traffic emanating from the web servers.

Figure 8
![image](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/b5183c15-aa12-4a96-82e0-f64fb7561d3f)

## Step 4: Creating a MYSQL Database

![image](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/1d56f4c1-bbb2-4742-9611-372bc7f05b58)

Putting a database in a private subnet enhances its security posture, reduces the risk of unauthorized access and data breaches, and helps organizations comply with regulatory requirements. To facilitate this process, we commence by establishing a database subnet group to accommodate our forthcoming database deployment (refer to Figure 9). Subsequently, we proceed with the creation of our database and its inclusion within the designated database subnet group (as illustrated in Figure 10).

Figure 9
![dbsubnets](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/74915534-cc31-49a4-84c4-e2ff35521662)

Figure 10
![image](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/8113a7c3-b36a-495f-ac89-bde71dab713e)

## Step 5: Attaching an EFS

The primary purpose of Amazon EFS is to provide scalable, elastic, and highly available file storage.

![efsdiagram](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/38ec656d-7f4a-4d3d-aa0e-75d4d2540329)

In our setup, we utilize EFS as the repository for our application code, specifically the WordPress installation, which our webservers will access. The first step involves creating an EFS with mount targets across all availability zones within our private subnet databases. Once the EFS setup is complete, we'll attach it to our EC2 instance.

Figure 11
![efs](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/18477997-ef5f-4322-919a-47930cfc1568)

## Step 6: Installing WordPress and Moving Files to EFS

To begin, we'll establish a security group enabling SSH access to our EC2 instance. Following this, adjustments to the EFS and Webserver Security Group rules will permit inbound traffic for SSH connections (Figure 8). Once configured, we'll proceed to launch the EC2 instance (Figure 12). After its deployment, we'll SSH into the instance to install WordPress. Subsequently, we'll transfer the WordPress files to our EFS. With this step completed, WordPress will be accessible from the EC2 instance.

Figure 12
![image](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/eb0dfd3b-9ae4-4f28-abaf-89f587d514c7)
