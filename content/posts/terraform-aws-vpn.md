---
title: "Automating AWS site-to-site VPNs with Terraform"
date: 2024-10-06T18:00:00-03:00
ShowToC: true
TocOpen: true
draft: false
cover:
    image: /images/terraform-aws-vpn/awsterraform1.png
    alt: 'Automating AWS site-to-site VPNs with Terraform'
    caption: 'Automating AWS site-to-site VPNs with Terraform'
---

# Introduction

In this post, I'll walk you through how to use Terraform to set up a VPN site-to-site connection on AWS, leveraging Infrastructure as Code (IaC) to make the process quicker, easier, and fully repeatable.

For the examples below, we’ll build a cloud infrastructure using a Virtual Private Gateway (VGW). In a future post, we’ll explore using a Transit Gateway (TGW) for more complex setups.

I’m running everything on a MacBook, but you can easily replicate these steps on a Linux jump host or any similar environment.



# Preparation

## AWS Identity and Access Management (IAM)

If you don't have one yet, the first step is to create an Access Key. This will allow you to authenticate and interact with AWS services through the CLI.

For this example, we’ll create a restrictive policy that only grants access to the specific resources needed, following the principle of least privilege.

1. Access the AWS console and go to IAM.
2. Under Access Management, go to Policies and click Create Policy.
3. I named my policy ```custom_policy_vpn_only```
3. In the policy editor, select JSON and copy/paste the JSON below:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:CreateVpnGateway",
                "ec2:CreateCustomerGateway",
                "ec2:CreateVpnConnection",
                "ec2:CreateVpnConnectionRoute",
                "ec2:DeleteVpnGateway",
                "ec2:DeleteCustomerGateway",
                "ec2:DeleteVpnConnection",
                "ec2:DeleteVpnConnectionRoute",
                "ec2:DescribeVpnGateways",
                "ec2:DescribeCustomerGateways",
                "ec2:DescribeVpnConnections",
                "ec2:AttachVpnGateway",
                "ec2:DetachVpnGateway",
                "ec2:CreateTags"
            ],
            "Resource": "*"
        }
    ]
}
```

4. Go back to the Access management menu, go to ***Users*** and click ***Create user***
5. Define the username as you wish. I created a user named "networkops""
6. On Set permissions, select Attach policies directly and select the policy we created before (use the search to make it easier to find the desired policy)
7. Once the user is created, click on the username and on the Summary page, click "Create access key"
8. For the Use case, select "Other"

Save your access key and secret access key


## Installing AWS CLI

The first step is to install AWS CLI. 

The procedure described below is available here: [AWS Docs](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

Download the file using the curl command. The -o option specifies the file name that the downloaded package is written to.
In this example, the file is written to ```AWSCLIV2.pkg``` in the current folder.

```
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
```

Run the standard macOS installer program, specifying the downloaded .pkg file as the source. Use the -pkg parameter to specify the name of the package to install, and the -target / parameter for which drive to install the package to. The files are installed to /usr/local/aws-cli, and a symlink is automatically created in /usr/local/bin. You must include sudo on the command to grant write permissions to those folders. 

```
sudo installer -pkg ./AWSCLIV2.pkg -target /
```

To verify that the shell can find and run the aws command in your $PATH, use the following commands:

```
which aws
```

```
aws --version
```



## Setting up AWS CLI profiles

Using AWS CLI profiles allows us to easily manage multiple sets of credentials, improving security and simplifying access to different AWS accounts and regions. This makes automating workflows and switching contexts more efficient and secure.

In this example we're going to set up a profile called ***terraform-vpn***, for lab purposes only.

Creating the profile:

```
aws configure --profile terraform-vpn
```

You will be prompted to enter your AWS credentials and configuration settings:
```
AWS Access Key ID [None]: <your access key id>
AWS Secret Access Key [None]: <your access key id>
Default region name [None]: <your aws region>
Default output format [None]: <I suggest using JSON as the output format>
```

Verify that the configuration was successful:
```
aws sts get-caller-identity --profile terraform-vpn
```

This command should return information about your AWS account, confirming that your credentials are working correctly.



# Diving into Automation

## Terraform configuration

I'm considering you already have Terraform installed. If you don't, the process is quite simple and there's a ton of content on the Internet on how to do it.

First, we're going to create a ```main.cf``` file:

```
terraform {
  required_version = ">= 0.12"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.0"
    }
  }
}

provider "aws" {
  region  = var.aws_region
  profile = var.aws_profile
}

# Create a Virtual Private Gateway (VGW)
resource "aws_vpn_gateway" "vgw" {
  vpc_id = var.vpc_id
  tags = {
    Name = var.vgw_name_tag
  }
}

# Create a Customer Gateway (CGW)
resource "aws_customer_gateway" "cgw" {
  bgp_asn    = var.bgp_asn
  ip_address = var.vpn_peer_ip
  type       = "ipsec.1"
  tags = {
    Name = var.cgw_name_tag
  }
}

# Create a VPN Connection with BGP (dynamic routing)
resource "aws_vpn_connection" "vpn_connection" {
  customer_gateway_id = aws_customer_gateway.cgw.id
  vpn_gateway_id      = aws_vpn_gateway.vgw.id
  type                = "ipsec.1"

# BGP will be used so this must be set to false, or removed from the project
# If you're using static routes, set this to true
  static_routes_only = false

  tags = {
    Name = var.vpn_connection_name_tag
  }
}
```

Now that we have a main.tf, we need to create two more files:

```variables.tf```

```
variable "aws_profile" {
  description = "AWS CLI profile to use"
  type        = string
}

variable "aws_region" {
  description = "AWS CLI profile to use"
  type        = string
}

variable "vpc_id" {
  description = "AWS region to use"
  type        = string
}

variable "vpn_peer_ip" {
  description = "Public IP address of the on-premises VPN peer"
  type        = string
}

variable "bgp_asn" {
  description = "BGP Autonomous System Number for the Customer Gateway"
  type        = number
}

variable "vgw_name_tag" {
  description = "Name tag for the Virtual Private Gateway"
  type        = string
}

variable "cgw_name_tag" {
  description = "Name tag for the Customer Gateway"
  type        = string
}

variable "vpn_connection_name_tag" {
  description = "Name tag for the VPN Connection"
  type        = string
}
```

And, lastly, our ```outputs.tf``` file. 

```
output "vpn_connection_id" {
  description = "VPN ID"
  value       = aws_vpn_connection.vpn_connection.id
}

output "tunnel1_inside_cidr" {
  description = "Inside CIDR block for tunnel 1"
  value       = aws_vpn_connection.vpn_connection.tunnel1_inside_cidr
}

output "tunnel1_aws_bgp_ip" {
  description = "AWS BGP peer IP for tunnel 1"
  value       = aws_vpn_connection.vpn_connection.tunnel1_vgw_inside_address
}

output "tunnel1_customer_bgp_ip" {
  description = "Customer gateway BGP peer IP for tunnel 1"
  value       = aws_vpn_connection.vpn_connection.tunnel1_cgw_inside_address
}

output "tunnel1_aws_public_ip" {
  description = "AWS public IP address for tunnel 1"
  value       = aws_vpn_connection.vpn_connection.tunnel1_address
}

output "tunnel2_inside_cidr" {
  description = "Inside CIDR block for tunnel 2"
  value       = aws_vpn_connection.vpn_connection.tunnel2_inside_cidr
}

output "tunnel2_aws_bgp_ip" {
  description = "AWS BGP peer IP for tunnel 2"
  value       = aws_vpn_connection.vpn_connection.tunnel2_vgw_inside_address
}

output "tunnel2_customer_bgp_ip" {
  description = "Customer gateway BGP peer IP for tunnel 2"
  value       = aws_vpn_connection.vpn_connection.tunnel2_cgw_inside_address
}

output "tunnel2_aws_public_ip" {
  description = "AWS public IP address for tunnel 2"
  value       = aws_vpn_connection.vpn_connection.tunnel2_address
}

output "tunnel1_preshared_key" {
  description = "Preshared key for tunnel 1"
  value       = aws_vpn_connection.vpn_connection.tunnel1_preshared_key
  sensitive   = true
}

output "tunnel2_preshared_key" {
  description = "Preshared key for tunnel 2"
  value       = aws_vpn_connection.vpn_connection.tunnel2_preshared_key
  sensitive   = true
}
```

At this point, your Terraform configuration should have the following structure:

```
├── main.tf
├── outputs.tf
└── variables.tf
```

## Initializing the Terraform Configuration

To initialize Terraform and download required providers:

```
terraform init
```


## Planning the changes

To preview the changes that Terraform will apply, we will use the ```terraform plan``` command, setting all the variables.

```
terraform plan \
  -var="aws_profile=terraform-vpn" \
  -var="aws_region=your_aws_region" \
  -var="vpc_id=your_vpc_id" \
  -var="vpn_peer_ip=your_on_prem_peer_public_ip" \
  -var="bgp_asn=your_on_prem_bgp_asn" \
  -var="vgw_name_tag=my-vpn-gateway" \
  -var="cgw_name_tag=my-customer-gateway" \
  -var="vpn_connection_name_tag=my-vpn-connection" \
  -out=vpn_to_onprem
```

This command will generate a detailed execution plan, and the resulting changes will be saved in a file named ```vpn_to_onprem```. Feel free to modify this file name as needed, depending on your preferences or environment.


## Applying the changes

Apply the planned changes to provision the infrastructure based on the output file generated by terraform plan:

```
terraform apply "vpn_to_onprem"
```

Once the command finishes running, Terraform will display some of the outputs defined in the ```outputs.tf``` file. However, you'll notice that sensitive information, such as pre-shared keys (PSKs), is not shown directly in the output for security reasons.


## Viewing Terraform Outputs

As mentioned above, the outputs defined in the ```outputs.tf``` file are automatically provided by Terraform and stored in the ```terraform.tfstate``` file. To view all outputs, including sensitive data, you have two options:

***Option 1***: Use the JSON flag
You can retrieve all outputs in JSON format, which will include sensitive information without redaction:
```
terraform output -json
```

***Option 2***: Manually Inspect the State File
Open the ```terraform.tfstate``` file and manually search for the sensitive data. This file contains the full configuration, including sensitive details like PSKs and IP addresses. Note that handling this file requires caution as it contains critical information.


## Terraform Destroy

One of Terraform's key benefits is how easily changes can be reverted with the ```terraform destroy``` command.

If you need to revert the changes we previously applied, use the following syntax:

```
terraform destroy \
  -var="aws_profile=terraform-vpn" \
  -var="aws_region=your_aws_region" \
  -var="vpc_id=your_vpc_id" \
  -var="vpn_peer_ip=your_on_prem_peer_public_ip" \
  -var="bgp_asn=your_on_prem_bgp_asn" \
  -var="vgw_name_tag=my-vpn-gateway" \
  -var="cgw_name_tag=my-customer-gateway" \
  -var="vpn_connection_name_tag=my-vpn-connection" \
```



# Configuring an on-prem Cisco Router

To wrap up, below is an example configuration script for a Cisco router. This script will establish the VPN tunnels to AWS and configure the necessary settings for BGP over IPsec.

In this example, I'm setting up only one of the VPN tunnels (AWS will create two for redundancy). To complete the configuration, simply replicate these settings using the information provided for the second tunnel.

***Some key points to remember:***
- AWS uses by default BGP ASN of 64512.
- When configuring BGP over an IPsec VPN connection on AWS, by default, AWS assigns two inside IP addresses per tunnel from a private IP range for each of the two tunnels that are created for high availability.

```
crypto isakmp policy 1
 encryption aes 128
 authentication pre-share
  group 2
  lifetime 28800
  hash sha

crypto keyring aws_vpn_keyring1
 pre-shared-key address <aws tunnel 1 public ip> key <psk for tunnel 1>

crypto isakmp profile aws_vpn_isakmp_profile1
  match identity address <aws tunnel 1 public ip>
  keyring aws_vpn_keyring1

crypto ipsec transform-set aws_vpn_tset esp-aes 128 esp-sha-hmac
 mode tunnel

crypto ipsec profile aws_vpn_ipsec_profile1
 set pfs group2
 set security-association lifetime seconds 3600
 set transform-set aws_vpn_tset

interface Tunnel1
 ip address <tunnel 1 private ip> 255.255.255.252
 ip tcp adjust-mss 1360
 ip virtual-reassembly
 tunnel source <your tunnel source ip/interface>
 tunnel destination <aws tunnel 1 public ip>
 tunnel mode ipsec ipv4
 tunnel protection ipsec profile aws_vpn_ipsec_profile1
 no shutdown
 exit

router bgp 65000
  neighbor <aws tunnel 1 private ip> remote-as 64512
  neighbor <aws tunnel 1 private ip> activate
  neighbor <aws tunnel 1 private ip> timers 10 30 30
  address-family ipv4 unicast
    neighbor <aws tunnel 1 private ip> remote-as 64512
    neighbor <aws tunnel 1 private ip> timers 10 30 30
    neighbor <aws tunnel 1 private ip> activate
    neighbor <aws tunnel 1 private ip> soft-reconfiguration inbound
```