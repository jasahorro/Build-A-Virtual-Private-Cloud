<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Build a Virtual Private Cloud

**Project Link:** [View Project](https://nextwork.ai/projects/7f907ac7-2473-5c15-a8be-edb36581cdba)

**Author:** Maria Jasmin Ahorro  
**Email:** jasminahorro23@gmail.com

---

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/7f907ac7-2473-5c15-a8be-edb36581cdba_2facf927)

## Introducing Today's Project!

In this project, I will demonstrate how to create and configure an Amazon VPC from scratch, including setting up a public subnet with auto-assigned IPv4 addresses and attaching an internet gateway. I'm doing this project to learn the core foundations of AWS networking and how to build a secure, customizable cloud network.

## Virtual Private Clouds (VPCs)

### What I did in this step

In this step, I will create a new Amazon VPC named NextWork VPC with an IPv4 CIDR block of 10.0.0.0/16 because I need an isolated virtual network in AWS to securely house and organize my cloud resources.

### How VPCs work

VPCs are logically isolated, custom virtual networks provisioned within an AWS Region that allow you to launch and manage cloud resources in a secure, defined environment. By creating a VPC, you gain complete control over your network architecture—including selecting your own IPv4 and IPv6 IP address ranges (CIDR blocks), dividing the network into public and private subnets across multiple Availability Zones, defining custom route tables, and deploying network components like internet gateways and security groups to precisely control how traffic flows between your resources and external networks. This explicit boundary ensures that your workloads remain private, preventing unauthorized external access while enabling seamless, encrypted communication between your interconnected services. Additionally, VPCs allow you to scale your infrastructure effortlessly, establish secure VPN connections to on-premises data centers, and enforce fine-grained traffic filtering using network access cont

### Why there is a default VPC in AWS accounts

There was already a default VPC in my account ever since my AWS account was created. This is because AWS automatically provisions a default VPC in every AWS Region to allow users to immediately launch and connect resources—like EC2 instances—without needing to manually design and configure a virtual network first.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/7f907ac7-2473-5c15-a8be-edb36581cdba_2facf927)

### Defining IPv4 CIDR blocks

To set up my VPC, I had to define an IPv4 CIDR block, which is a range of IP addresses assigned to the network that determines how many internal IP addresses are available for resources created within the VPC.

## Subnets

### What I did in this step

In this step, I will create a subnet named Public 1 inside my NextWork VPC with an IPv4 CIDR block of 10.0.0.0/24 because I need to partition my VPC into smaller network segments to organize and isolate resources.

### Creating and configuring subnets

Subnets are smaller subdivisions or network segments created within a Virtual Private Cloud (VPC) that allow you to group, organize, and isolate cloud resources based on operational and security needs. By dividing a VPC's larger IP address space into individual subnets, you can establish distinct security boundaries—such as placing public-facing web servers in public subnets with direct access to the internet, while isolating sensitive databases in private subnets with restricted entry. Each subnet resides within a single Availability Zone and cannot span across multiple zones, ensuring that localized infrastructure failures are contained.

There are already subnets existing in my account, one for every Availability Zone in the default VPC that AWS automatically provisions upon account creation. AWS sets up these default subnets so that you can immediately launch EC2 instances and deploy workloads without needing to manually design a network architecture from scratch. Each default subn

### Public vs private subnets

The difference between public and private subnets are that public subnets have direct access to and from the public internet, whereas private subnets are isolated from direct internet traffic to secure sensitive internal resources.For a subnet to be considered public, it has to be attached to an internet gateway (via a route table entry pointing $0.0.0.0/0$ to the IGW) and have auto-assign public IPv4 addresses enabled for resources launched inside it.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/7f907ac7-2473-5c15-a8be-edb36581cdba_157c4219)

### Auto-assigning public IPv4 addresses

Once I created my subnet, I enabled the "auto-assign public IPv4 address" setting. This setting makes sure that any EC2 instance launched inside this subnet automatically receives a public IPv4 address so that it can directly communicate with external networks and the public internet without needing manual IP allocation.

## Internet gateways

### What I did in this step

In this step, I will create an internet gateway named NextWork IG and attach it to my NextWork VPC because I need to enable a connection between my private network and the public internet so that resources in my VPC can send and receive external traffic.

### Setting up internet gateways

Internet gateways are horizontally scaled, redundant, and highly available VPC components that allow communication between resources inside your AWS VPC and the public internet. They serve two primary purposes: providing a target in your VPC route tables for internet-routable traffic, and performing Network Address Translation (NAT) for instances that have been assigned public IPv4 addresses.

By attaching an internet gateway to your VPC, you enable resources in public subnets—such as web servers or application load balancers—to connect directly to external services and allow users from the outside world to reach your hosted applications. The internet gateway seamlessly maps public IP addresses to internal private IP addresses without imposing bandwidth constraints or creating single points of failure. Without an internet gateway attached to your VPC and referenced in your subnet's route table, even instances with public IP addresses remain isolated from external networks and completel

Attaching an internet gateway to a VPC means linking your isolated virtual network to the public internet, enabling bidirectional communication between your VPC resources and external networks. If I missed this step, my EC2 instances and resources inside the VPC would remain completely isolated from the outside world—they wouldn't be able to receive incoming internet traffic or reach external web services, even if they have public IP addresses and public subnet configurations.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/7f907ac7-2473-5c15-a8be-edb36581cdba_4ae90410)

## Using the AWS CLI

### What I'm doing in this extension

In this project extension, I will configure custom route table rules for Public 1 to route outbound traffic directly to NextWork IG because attaching an internet gateway is only the first step—resources inside the subnet also need an explicit routing rule pointing 0.0.0.0/0 to the gateway to become fully accessible to the public internet.

### Exploring CloudShell and CLI

VPC resources could also be created with CloudShell, which is a browser-based, pre-authenticated terminal environment built directly into the AWS Management Console that allows you to manage cloud resources using command-line tools without needing local setup. CLI is the Command Line Interface, a text-based tool that lets you interact with and automate AWS services by executing terminal commands instead of clicking through the graphical web interface.

### Debugging my setup

To set up a VPC or a subnet, you can use the command aws ec2 create-vpc or aws ec2 create-subnet. Make sure to avoid errors by including all required parameters, such as --vpc-id and --cidr-block (e.g., 10.0.0.0/24), as omitting these required arguments will trigger a MissingParameters error.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/7f907ac7-2473-5c15-a8be-edb36581cdba_9b2465411)

### Comparing CloudShell vs AWS Console

Compared to using the AWS Console, an advantage of using commands is speed, automation, and repeatably executing provisioning tasks without clicking through multiple menus. An advantage of using the Console is its visual layout and interactive menus, which make it easy to verify network structures and resource states at a glance. Overall, I preferred using the AWS Console because I encountered a temporary account verification hold with CloudShell, and navigating through the visual interface allowed me to complete the project smoothly without waiting for terminal access.

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/7f907ac7-2473-5c15-a8be-edb36581cdba)*
