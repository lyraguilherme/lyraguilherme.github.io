---
title: "Automating AWS site-to-site VPNs with Terraform"
date: 2024-10-06T18:00:00-03:00
ShowToC: true
TocOpen: true
draft: false
cover:
    image: /terraform-aws-vpn/awsterraform1.png
    alt: 'Automating AWS site-to-site VPNs with Terraform'
    caption: 'Automating AWS site-to-site VPNs with Terraform'
tags:
 - Automation
 - AWS
 - IaC
 - Terraform
 - VPN
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

```json
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

```shell
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
```

Run the standard macOS installer program, specifying the downloaded .pkg file as the source. Use the -pkg parameter to specify the name of the package to install, and the -target / parameter for which drive to install the package to. The files are installed to /usr/local/aws-cli, and a symlink is automatically created in /usr/local/bin. You must include sudo on the command to grant write permissions to those folders. 

```shell
sudo installer -pkg ./AWSCLIV2.pkg -target /
```

To verify that the shell can find and run the aws command in your $PATH, use the following commands:

```shell
which aws
```

```shell
aws --version
```



## Setting up AWS CLI profiles

Using AWS CLI profiles allows us to easily manage multiple sets of credentials, improving security and simplifying access to different AWS accounts and regions. This makes automating workflows and switching contexts more efficient and secure.

In this example we're going to set up a profile called ***terraform-vpn***, for lab purposes only.

Creating the profile:

```shell
aws configure --profile terraform-vpn
```

You will be prompted to enter your AWS credentials and configuration settings:
```shell
AWS Access Key ID [None]: <your access key id>
AWS Secret Access Key [None]: <your access key id>
Default region name [None]: <your aws region>
Default output format [None]: <I suggest using JSON as the output format>
```

Verify that the configuration was successful:
```shell
aws sts get-caller-identity --profile terraform-vpn
```

This command should return information about your AWS account, confirming that your credentials are working correctly.



# Diving into Automation

![Diving into automation](neom-n7SdozgzvyA-unsplash.jpg)

Photo by [NEOM](https://unsplash.com/@neom?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/photos/a-person-swimming-in-a-deep-blue-ocean-n7SdozgzvyA?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)


## Terraform configuration

I'm considering you already have Terraform installed. If you don't, the process is quite simple and there's a ton of content on the Internet on how to do it.

First, we're going to create a ```main.cf``` file:

```shell
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

```shell
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

And, lastly, an ```outputs.tf``` file. 

```shell
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

```shell
├── main.tf
├── outputs.tf
└── variables.tf
```



## Initializing the Terraform Configuration

To initialize Terraform and download required providers:

```shell
terraform init
```


## Planning the changes

To preview the changes that Terraform will apply, we will use the ```terraform plan``` command, setting all the variables.

```shell
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

```shell
terraform apply "vpn_to_onprem"
```

Once the command finishes running, Terraform will display some of the outputs defined in the ```outputs.tf``` file. However, you'll notice that sensitive information, such as pre-shared keys (PSKs), is not shown directly in the output for security reasons.



## Viewing Terraform Outputs

As mentioned above, the outputs defined in the ```outputs.tf``` file are automatically provided by Terraform and stored in the ```terraform.tfstate``` file. To view all outputs, including sensitive data, you have two options:

***Option 1***: Use the JSON flag
You can retrieve all outputs in JSON format, which will include sensitive information without redaction:
```shell
terraform output -json
```

***Option 2***: Manually Inspect the State File
Open the ```terraform.tfstate``` file and manually search for the sensitive data. This file contains the full configuration, including sensitive details like PSKs and IP addresses. Note that handling this file requires caution as it contains critical information.



## Terraform Destroy

One of Terraform's key benefits is how easily changes can be reverted with the ```terraform destroy``` command.

If you need to revert the changes we previously applied, use the following syntax:

```shell
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


# Configuring our VPN peer (Cisco Router)

To wrap up, below is a Python script I created. This script generates the VPN and BGP configuration for a Cisco router based on Terraform outputs. Please use with caution and always double-check before applying it in a production environment.

***Key points:***
- AWS uses by default BGP ASN 64512
- On my lab environment I'm using BGP ASN 65000 on the VPN peer

Both of the settings mentioned above are hardcoded on the script. Change as needed.

```python
import subprocess
import json

def get_terraform_outputs():
    try:
        # Get the Terraform output in JSON format
        result = subprocess.run(
            ["terraform", "output", "-json"],
            capture_output=True,
            text=True,
            check=True
        )
        return json.loads(result.stdout)
    except subprocess.CalledProcessError as e:
        print(f"Error: {e}")
        return {}

def generate_cisco_config(outputs):

    # Sets variables based on Terraform outputs
    tunnel1_aws_bgp_ip = outputs.get("tunnel1_aws_bgp_ip", {}).get("value", "")
    tunnel2_aws_bgp_ip = outputs.get("tunnel2_aws_bgp_ip", {}).get("value", "")
    tunnel1_customer_bgp_ip = outputs.get("tunnel1_customer_bgp_ip", {}).get("value", "")
    tunnel2_customer_bgp_ip = outputs.get("tunnel2_customer_bgp_ip", {}).get("value", "")    
    tunnel1_aws_public_ip = outputs.get("tunnel1_aws_public_ip", {}).get("value", "")
    tunnel2_aws_public_ip = outputs.get("tunnel2_aws_public_ip", {}).get("value", "")    
    tunnel1_preshared_key = outputs.get("tunnel1_preshared_key", {}).get("value", "")
    tunnel2_preshared_key = outputs.get("tunnel2_preshared_key", {}).get("value", "")

    # Cisco config block
    config = f'''

    !!! DOUBLE-CHECK BEFORE APPLYING IN PRODUCTION !!!
    
    crypto isakmp policy 1
        encryption aes 128
        authentication pre-share
        group 2
        lifetime 28800
        hash sha

    crypto keyring aws_vpn_keyring1
        pre-shared-key address {tunnel1_aws_public_ip} key {tunnel1_preshared_key}

    crypto keyring aws_vpn_keyring2
        pre-shared-key address {tunnel2_aws_public_ip} key {tunnel2_preshared_key}        

    crypto isakmp profile aws_vpn_isakmp_profile1
        match identity address {tunnel1_aws_public_ip}
        keyring aws_vpn_keyring1

    crypto isakmp profile aws_vpn_isakmp_profile2
        match identity address {tunnel2_aws_public_ip}
        keyring aws_vpn_keyring2        
        
    crypto ipsec transform-set aws_vpn_transform_set esp-aes 128 esp-sha-hmac
        mode tunnel

    crypto ipsec profile aws_vpn_ipsec_profile
        set pfs group2
        set security-association lifetime seconds 3600
        set transform-set aws_vpn_transform_set

    interface Tunnel1
        ip address {tunnel1_customer_bgp_ip} 255.255.255.252
        ip tcp adjust-mss 1360
        ip virtual-reassembly
        tunnel source <your tunnel source ip/interface>
        tunnel destination {tunnel1_aws_public_ip}
        tunnel mode ipsec ipv4
        tunnel protection ipsec profile aws_vpn_ipsec_profile
        no shutdown
        exit

    interface Tunnel2
        ip address {tunnel2_customer_bgp_ip} 255.255.255.252
        ip tcp adjust-mss 1360
        ip virtual-reassembly
        tunnel source <your tunnel source ip/interface>
        tunnel destination {tunnel2_aws_public_ip}
        tunnel mode ipsec ipv4
        tunnel protection ipsec profile aws_vpn_ipsec_profile
        no shutdown
        exit        

    router bgp 65000
        neighbor {tunnel1_aws_bgp_ip} remote-as 64512
        neighbor {tunnel1_aws_bgp_ip} activate
        neighbor {tunnel1_aws_bgp_ip} timers 10 30 30
        neighbor {tunnel2_aws_bgp_ip} remote-as 64512
        neighbor {tunnel2_aws_bgp_ip} activate
        neighbor {tunnel2_aws_bgp_ip} timers 10 30 30
        address-family ipv4 unicast
            neighbor {tunnel1_aws_bgp_ip} remote-as 64512
            neighbor {tunnel1_aws_bgp_ip} timers 10 30 30
            neighbor {tunnel1_aws_bgp_ip} activate
            neighbor {tunnel1_aws_bgp_ip} soft-reconfiguration inbound
            neighbor {tunnel2_aws_bgp_ip} remote-as 64512
            neighbor {tunnel2_aws_bgp_ip} timers 10 30 30
            neighbor {tunnel2_aws_bgp_ip} activate
            neighbor {tunnel2_aws_bgp_ip} soft-reconfiguration inbound
            
        end

    !!! DOUBLE-CHECK BEFORE APPLYING IN PRODUCTION !!!

    '''

    return (config)

if __name__ == "__main__":
    outputs = get_terraform_outputs()
    if outputs:
        cisco_config = generate_cisco_config(outputs)
        print(cisco_config)
```
