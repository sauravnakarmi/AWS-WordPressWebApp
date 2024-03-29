## Overview
This project aims to design and implement a robust, multi-tier WordPress web application on Amazon Web Services (AWS), incorporating Virtual Private Cloud (VPC), Elastic Compute Cloud (EC2), Application Load Balancer (ALB), Relational Database Service (RDS), and Elastic File System (EFS). The primary goal is to achieve high availability and fault tolerance while maintaining scalability and security. I refrenced AOS notes tutorial when building this architecture (@AOS Note is his youtube channel). 

## Challenges Faced
Building a three-tier web application with AWS can present several challenges due to the complexity of the services involved. A major issue I faced when tackling this project was figuring out what the purpose of some of the services were. For example, I was confused what the inclusion of EFS did for the system as a whole. I later realized that the EFS was used to offset the time it takes to spin up new EC2 instances since the wordpress files could be directly taken from the EFS. Another example of this confusion can be seen with the public subnets. At first, I thought the public subnets would be where the users traffic would be sent to, but the traffic actually gets sent to the application load balancer which then distributes it to the EC2 instances in the network. I also had trouble SSHing into the EC2 instance since there were various security group permissions that had to be set and the right key pair had to be used. 

## Final Product
![finishedinfrastructure](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/94fa7c85-38fc-412b-ad5a-deb5c3d0c4e1)

## Steps

### Step 1: Building a Secure Network - Three Tier Application Infrastructure

"The multi-tier architecture pattern provides a general framework to ensure decoupled and independently scalable application components can be separately developed, managed, and maintained" (from the AWS website). 

![WebApplicationStage1](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/092605ba-29ba-4218-a83c-ee35cc594b05)

The initial phase of crafting a WordPress web application involves network setup. Commencing with the creation of a VPC within the us-east-1 region, as illustrated in Figure 1, I proceeded by adding an internet gateway to provide internet connectivity to the public subnets, depicted in Figure 2. Subsequently, I delineated both public and private subnets, as showcased in Figure 3. The public subnets will serve as place for the NAT gateways to access the internet. Meanwhile, the first and second private subnets constitute the application tier, managing traffic from the application load balancer and housing EC2 instances with the WordPress installation. The third and fourth subnets form the database tier, housing critical data such as WordPress files. To ensure proper internet access for the public subnets and efficient routing of VPC traffic to the private subnets, I created route tables. This included establishing a public route table, with an internet gateway to allow for internet access to the public subnet, alongside a private route table with no internet gateway, as depicted in Figure 4. These route tables were subsequently linked to their respective public and private subnets.

Figure 1
![vpc](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/5606d210-5c04-4380-8572-d6021c8e1e8f)

Figure 2
![internetgateway](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/0513c804-2b9d-4aec-8aa3-59f3b442f4d9)

Figure 3
![Subnets](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/c9290c54-6b80-4cb9-8673-090ca91973c6)

Figure 4
![RoutTables](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/daa7390b-444c-4943-805e-f3ffda44d709)

### Step 2: Attaching a NAT Gateway

NAT Gateways serve a crucial role in granting internet access to the application tier while safeguarding EC2 instances from direct user access, thereby adhering to security best practices. This setup allows EC2 instances to download patches without potential malicious user intervention.

![NatGatewayDiagram](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/93b0a368-5f3b-4203-9f32-2fdb7a4d2363)

Initiating the process, I provisioned two Elastic IP addresses designated for NAT Gateways, as delineated in Figure 5. Subsequently, the NAT Gateways were instantiated and associated with their respective Elastic IP addresses, as depicted in Figure 6. To ensure smooth traffic routing from the EC2 instances to the NAT Gateways, I configured route tables within the private subnet, establishing routes directing traffic to the NAT Gateways, as outlined in Figure 7.

Figure 5
![ElasticIp](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/3ba2c18d-ee70-4c91-987d-a422c2e85d87)

Figure 6
![NatGateway](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/921aa575-fcbe-4483-accb-42dd9405851a)

Figure 7
![NatGatewayRouteTable](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/d8c9a1a3-4267-4ded-bafc-5468d8280562)

### Step 3: Controlling Traffic with Security Groups

To ensure seamless traffic routing within the web application (as depicted in Figure 8), four primary security groups are imperative. The initial security group facilitates the ingress of traffic from the internet into the application load balancer. Subsequently, the second security group allows traffic exclusively from the application load balancer to access the web servers. Furthermore, the third security group makes it so that the database servers solely accept traffic from the web servers. Lastly, the EFS is configured to solely accept traffic from the web servers.

Figure 8
![image](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/b5183c15-aa12-4a96-82e0-f64fb7561d3f)

### Step 4: Creating a MYSQL Database

Putting a database in a private subnet enhances its security posture, reduces the risk of unauthorized access and data breaches, and helps organizations comply with regulatory requirements.

![image](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/1d56f4c1-bbb2-4742-9611-372bc7f05b58)

 To facilitate this process, we start by establishing a database subnet group to accommodate our database deployment (refer to Figure 9). Subsequently, we proceed with the creation of our database and add it to its designated database subnet group (as illustrated in Figure 10).

Figure 9
![dbsubnets](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/74915534-cc31-49a4-84c4-e2ff35521662)

Figure 10
![image](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/8113a7c3-b36a-495f-ac89-bde71dab713e)

### Step 5: Attaching an EFS

The primary purpose of Amazon EFS is to provide scalable, elastic, and highly available file storage.

![efsdiagram](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/38ec656d-7f4a-4d3d-aa0e-75d4d2540329)

In our setup, we utilize EFS as the repository for our application code, specifically the WordPress installation, which our webservers will access. The first step involves creating an EFS with mount targets across two availability zones within our private subnet databases. Once the EFS setup is complete, we'll attach it to our EC2 instance.

Figure 11
![efs](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/18477997-ef5f-4322-919a-47930cfc1568)

### Step 6: Installing WordPress and Moving Files to EFS

![FinalDiagram](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/4475e295-5baa-4816-81cc-a74941905f65)

To begin, we'll establish a security group enabling SSH access to our EC2 instance. Following this, adjustments to the EFS and Webserver Security Group rules will permit inbound traffic for SSH connections (Figure 8). Once configured, we'll proceed to launch the EC2 instance (Figure 12). After its deployment, we'll SSH into the instance to install WordPress. Subsequently, we'll transfer the WordPress files to our EFS. With this step completed, WordPress will be accessible from the EC2 instance.

Figure 12
![image](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/eb0dfd3b-9ae4-4f28-abaf-89f587d514c7)

### Step 7: Application Load Balancer

![finishedinfrastructure](https://github.com/sauravnakarmi/AWS-WordPressWebApp/assets/70821330/a91c41ad-1354-4e9b-9251-6df05f3e6cf5)

In order to have a properly functioning application load balancer, multiple EC2 instances are required. So, we start by creating a new EC2 instance. However, this time we have transfered the Wordpress files to our EFS so the deployment of the second EC2 instance will be much quicker. Once the EC2 instances have been created we can start with the creation of the application load balancer, making sure to add the two EC2 instances we have created into the target group. With that we should be able to access the WebApp from the application load balancer link.

## Conclusion
The inital goal of this project was to implement as many AWS technologies into a project as I could, in an effort to familiarize myself with AWS as a platform. This project helped me gain a deeper understanding of cloud computing concepts, such as infrastructure as a service, platform as a service, and software as a service. Designing a three-tiered web application requires thoughtlful consideration of architecture components, including frontend, backend, and databse layers. I learned how to architect scalable, reliable, and secure applications using AWS services like Amazon EC2 to host web servers, Amazon RDS for managing databases and Amazon Route 53 for DNS management. I also gained experinece in deploying and managing applications in a cloud environment using the AWS Management console. Along with managing permissions and security with Security groups, routing tables, public and private subnets. One of the benefits of using AWS is its ability to scale resources based on demand. This was utilized with the use of an application load balancer which allowed traffic to be evenly distributed. Another feature that AWS offers is fault tolerance and high availibility. I used mult-az deployment to make use of both of these features. Security is a top priority when deploying applications in the cloud. I learned about AWS security practices including network security, compliance, and routing network traffic. Overall, creating a three-tiered web application in AWS provided me with valuable hands-on experience with cloud techonologies and prepared me for designing, deploying, and managing modern web applications in a scalable and cost-effective manner. 

