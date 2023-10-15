# AWS CLOUD SOLUTION FOR 2 COMPANY WEBSITES USING A REVERSE PROXY TECHNOLOGY

**In implementing this project, we will manually build a secure infrastructure inside AWS VPC (Virtual Private Cloud) network for a fictitious company (we'll chose any name for it) that uses WordPress CMS for its main business website, and a Tooling Website for their DevOps team.  
As part of the company’s desire for improved security and performance, a decision has been made to use a reverse proxy technology from [NGINX](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/) to achieve this.**

Cost, Security, and Scalability are the major requirements for this project. Hence, implementing the **architecture designed below**, ensures that the infrastructure for both websites (WordPress and Tooling) is resilient to Web Server’s failures, can accommodate to increased traffic and, at the same time, has reasonable cost.

![Alt text](1.aws-multiple-architecture.png)

#### SET UP A VIRTUAL PRIVATE NETWORK (VPC)

Ensure to always make reference to the architectural diagram and ensure that your configuration is aligned with it.

1. Create a VPC and enable DNS hostname

![Alt text](1.create-vpc.png)

![Alt text](1.create-vpc3.png)

![Alt text](2.enable-DNS-hostname.png)

2. Create an Internet Gateway (*igw*)

![Alt text](2.create-igw.png)

![Alt text](2.create-igw2.png)

3. Create subnets as shown in the architecture

![Alt text](2.create-subnet.png)

![Alt text](2.create-subnet1.png)

![Alt text](2.create-subnet2.png)

so far, we now have 2 public subnets and 4 private subnets

4. Create a route table (*rtb*) and associate it with the public subnets

create route table for public and private subnets

![Alt text](1.create-rtb.png)


5. Create a route table and associate it with the private subnets

![Alt text](1.associate-rtb.png)

![Alt text](1.associate-rtb2.png)

![Alt text](1.associate-rtb3.png)



6. Edit a route in public route table, and associate it with the Internet Gateway. (This is what allows a public subnet to be accisble from the Internet)
Note: that the route in the private route table will only be edited and associaated with the NAT Gateway.

![Alt text](1.edit-routes.png)

![Alt text](1.edit-routes2.png)

7. Create 3 Elastic IPs

![Alt text](1.allocate-eip.png)

8. Create a Nat Gateway and assign one of the Elastic IPs (*The other 2 will be used by Bastion hosts)

![Alt text](1.nat-gateway-eip-attatch.png)

9. Create a Security Group for:

- Nginx Servers: Access to Nginx should only be allowed from a Application Load balancer (ALB). At this point, we have not created a load balancer, therefore we will update the rules later. For now, just create it and put some dummy records as a place holder.

- Bastion Servers: Access to the Bastion servers should be allowed only from workstations that need to SSH into the bastion servers. Hence, you can use your workstation public IP address. To get this information, simply go to your terminal and type curl www.canhazip.com

- External Application Load Balancer: This ALB will be available from the Internet

- Internal Application Load Balancer: To allow access from the Nginx reverse proxy server

- Webservers: Access to Webservers should only be allowed from the Nginx servers. Since we do not have the servers created yet, just put some dummy records as a place holder, we will update it later.

- Data Layer: Access to the Data layer, which is comprised of Amazon Relational Database Service (RDS) and Amazon Elastic File System (EFS) must be carefully desinged – only webservers should be able to connect to RDS, while Nginx and Webservers will have access to EFS Mountpoint.

The Bastion server will also need to have ssh access to the database

![Alt text](1.all-security-grps.png)


#### TLS Certificates From Amazon Certificate Manager (ACM)
You will need TLS certificates to handle secured connectivity to your Application Load Balancers (ALB).

- Navigate to AWS ACM

![Alt text](1.aws-certificate-manager.png)

- Request a public wildcard certificate for the domain name you registered in any prefered domain name registry

- Use DNS to validate the domain name

- Tag the resource


![Alt text](1.aws-certificate-manager2.png)

![Alt text](1.aws-certificate-manager3.png)

![Alt text](1.aws-certificate-manager4.png)

![Alt text](1.aws-certificate-manager-issued.png)

![Alt text](1.aws-certificate-manager-issued2.png)

CREATE AND SETUP EFS

Amazon Elastic File System (Amazon EFS) provides a simple, scalable, fully managed elastic Network File System (NFS) for use with AWS Cloud services and on-premises resources. In this project, we will utulize EFS service and mount filesystems on both Nginx and Webservers to store data.

1. Create an EFS filesystem

![Alt text](1.efs-create.png)

![Alt text](1.efs-create1.png)

2. Create an EFS mount target per AZ in the VPC, associate it with both subnets dedicated for data layer

3. Associate the Security groups created earlier for data layer.

![Alt text](1.efs-create2.png)

4. Create an EFS access point. (Give it a name and leave all other settings as default)

![Alt text](1.efs-create-accesspoint.png)

![Alt text](1.efs-create-accesspoint2.png)

![Alt text](1.efs-create-accesspoint3.png)