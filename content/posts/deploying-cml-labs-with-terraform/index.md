---
title: 'Deploying a Cisco CML lab with Terraform'
date: 2026-11-01 10:00:00-03:00
draft: false
tags:
   - Automation
   - Cisco
   - CML
   - IaC
   - Terraform
showTableOfContents: true
featureAlt: "Deploying a Cisco CML lab with Terraform"
summary: "Using Terraform and the CML2 provider to deploy a complete Cisco Modeling Labs topology as code, from nodes and links to day-0 configs, in a single terraform apply."
---

# Introduction

Infrastructure as Code (IaC) has been part of the cloud world for years, and it's now becoming more and more popular in network labs too. Instead of dragging nodes around a canvas and pasting configs device by device, we describe the whole lab in text files, keep it in Git, and let a tool build it for us. The lab becomes something we can version, review, share, tear down, and rebuild exactly the same way whenever we need it.

In this post, I'll walk you through how to use Terraform to deploy a complete Cisco Modeling Labs (CML) topology. The focus here is the deployment itself: creating the lab, the nodes, the links, injecting day-0 configs, and starting everything with a single ```terraform apply```.

The topology we're building is an enterprise network with an HQ, two branches, an MPLS L3VPN provider, and an Internet provider. The day-0 configs bring up the full set of lab technologies: MPLS TE tunnels with fast reroute, IS-IS LFA and remote LFA, Internet access with NAT, and a GRE overlay that takes over when a branch loses MPLS. I'll dig into each of them in future posts.

You can easily replicate these steps on a Linux jump host or any similar environment.

***My environment:***
- Cisco Modeling Labs 2.10
- Terraform 1.15.5
- CiscoDevNet/cml2 Terraform provider 0.10.1
- IOL images only (```iol-xe``` and ```ioll2-xe```, version 17.18.02)

The full project is available on my GitHub: https://github.com/lyraguilherme/cisco-cml-iac

------

# The topology

{{< mermaid >}}
flowchart LR
    subgraph HQ["HQ (AS 65100)"]
        HQSW["HQ-SW"]
        HQR1["HQ-RTR1"]
        HQR2["HQ-RTR2"]
    end
    subgraph MPLS["MPLS provider (AS 65000)"]
        PE1["PE1"]
        P1["P1"]
        P2["P2"]
        P3["P3"]
        P4["P4"]
        PE2["PE2"]
        PE3["PE3"]
    end
    subgraph INET["Internet provider"]
        ISP["ISP"]
    end
    subgraph SITE1["Branch 1 (AS 65101)"]
        B1R1["BR1-RTR1"]
        B1R2["BR1-RTR2"]
        B1SW["BR1-SW"]
    end
    subgraph SITE2["Branch 2 (AS 65102)"]
        B2R1["BR2-RTR1"]
        B2R2["BR2-RTR2"]
        B2SW["BR2-SW"]
    end
    HQSW --- HQR1
    HQSW --- HQR2
    HQR1 --- PE1
    HQR2 --- PE1
    PE1 --- P1
    PE1 --- P2
    P1 --- P2
    P1 --- P3
    P2 --- P4
    P1 --- P4
    P3 --- P4
    P3 --- PE2
    P4 --- PE3
    PE2 --- B1R1
    PE3 --- B2R1
    B1R1 --- B1SW
    B1R2 --- B1SW
    B2R1 --- B2SW
    B2R2 --- B2SW
    HQR1 --- ISP
    HQR2 --- ISP
    B1R2 --- ISP
    B2R2 --- ISP
    %% Internet links (the last four, 0-based indexes 19 to 22) in orange
    linkStyle 19,20,21,22 stroke:#ea580c,stroke-width:4px
    %% PE-CE links (0-based indexes 2, 3, 13 and 14) in blue
    linkStyle 2,3,13,14 stroke:#2563eb,stroke-width:4px
{{< /mermaid >}}

Thick blue lines are PE-CE links, thick orange lines are Internet links, and the thin lines are MPLS core and site LAN links.

***Key points:***
- HQ has two edge routers, each one connected to both the MPLS provider (PE1) and the Internet provider (ISP), plus an L3 switch
- Each branch has two routers, RTR1 connected to MPLS and RTR2 connected to the Internet, plus an L3 switch
- The MPLS core is a ring of four P routers (P1-P2-P4-P3) with a P1-P4 diagonal, which gives us multiple paths for traffic engineering and interesting LFA coverage
- The provider runs IS-IS as IGP, with LDP, MPLS TE, and RSVP enabled on every core link, TE tunnels between the PEs, and LFA protecting the IGP
- Each site runs OSPFv2 internally and eBGP as PE-CE protocol
- The ISP sends a default route to the Internet-facing routers, which NAT their site traffic
- MPLS is the primary WAN, and a GRE overlay over the Internet between the branches and HQ is the backup
- That's 17 nodes and 23 links in total, and IOL is light enough to run all of them without much trouble

------

# Preparation

## Requirements

I'm considering you already have Terraform installed and a CML controller reachable from your machine. You'll also need an account on CML with permission to create labs.

Clone the repository:

```shell
git clone https://github.com/lyraguilherme/cisco-cml-iac
cd cisco-cml-iac
```

At this point, you should have the following structure:

```shell
├── README.md
├── configs
│   ├── BR1-RTR1.cfg
│   ├── ...
│   └── PE3.cfg
├── main.tf
├── outputs.tf
├── providers.tf
├── terraform.tfvars.example
├── variables.tf
└── versions.tf
```


## Setting up credentials

The CML address and credentials live in a ```terraform.tfvars``` file, which Terraform loads automatically. The repository only ships a ```terraform.tfvars.example``` with placeholder values, and the real file is listed under ```.gitignore```, so your credentials never end up in Git.

```shell
cp terraform.tfvars.example terraform.tfvars
```

Edit the file with your own values:

```hcl
cml_address     = "https://cml.example.local"
cml_username    = "admin"
cml_password    = "changeme"
cml_skip_verify = true
```

> **Note:** ```cml_skip_verify = true``` disables TLS certificate verification, which is fine for a lab controller with a self-signed certificate. Keep in mind that the password is stored in plain text on your machine, so this approach is meant for lab use only.

------

# The Terraform project

## Provider configuration

First, ```versions.tf``` pins Terraform and the CML2 provider:

```hcl
terraform {
  required_version = ">= 1.5"

  required_providers {
    cml2 = {
      source  = "CiscoDevNet/cml2"
      version = "~> 0.10"
    }
  }
}
```

Then ```providers.tf``` points the provider to our controller, using the variables from ```terraform.tfvars```:

```hcl
provider "cml2" {
  address     = var.cml_address
  username    = var.cml_username
  password    = var.cml_password
  skip_verify = var.cml_skip_verify
}
```

The password variable is marked as ```sensitive``` in ```variables.tf```, so Terraform hides it from the plan and apply output:

```hcl
variable "cml_password" {
  description = "CML password"
  type        = string
  sensitive   = true
}
```

The image versions are variables too, so moving the whole lab to a different IOL release is a one-line change:

```hcl
variable "iol_router_image" {
  description = "Image definition for iol-xe routers"
  type        = string
  default     = "iol-xe-17-18-02"
}
```


## Describing the topology as data

This is possibly the most interesting part of the project. Instead of writing one resource block per device, the whole topology is described as two maps in ```main.tf```: one for nodes and one for links.

Each node has a node definition, an image, canvas coordinates, and tags:

```hcl
locals {
  nodes = {
    "HQ-SW"   = { nodedefinition = "ioll2-xe", image = var.iol_switch_image, x = -800, y = 0, tags = ["hq"] }
    "HQ-RTR1" = { nodedefinition = "iol-xe", image = var.iol_router_image, x = -600, y = -100, tags = ["hq"] }
    "PE1"     = { nodedefinition = "iol-xe", image = var.iol_router_image, x = -400, y = 0, tags = ["mpls"] }
    "P1"      = { nodedefinition = "iol-xe", image = var.iol_router_image, x = -200, y = -100, tags = ["mpls"] }
    # ...
  }
```

Each link connects two nodes, with the interface slot on each side:

```hcl
  links = {
    "hqsw-hqrtr1" = { a = "HQ-SW", slot_a = 0, b = "HQ-RTR1", slot_b = 0 }
    "hqrtr1-pe1"  = { a = "HQ-RTR1", slot_a = 1, b = "PE1", slot_b = 0 }
    "pe1-p1"      = { a = "PE1", slot_a = 2, b = "P1", slot_b = 0 }
    # ...
  }
}
```

On IOL, slot N maps to ```Ethernet(N/4)/(N%4)```. In practice, slots 0 to 3 are ```Ethernet0/0``` to ```Ethernet0/3```, slot 4 is ```Ethernet1/0```, and so on. Setting the slots explicitly is important, since it guarantees each cable lands on the interface our configs expect.

To keep things consistent, every site router uses the same interface roles:
- ```Ethernet0/0``` connects to the site switch
- ```Ethernet0/1``` connects to MPLS
- ```Ethernet0/2``` connects to the Internet

Adding a new router or a new link to the lab is now just one more line in one of these maps.


## Creating the lab, nodes, and links

With the data in place, the resources themselves are short. The lab is a single resource:

```hcl
resource "cml2_lab" "this" {
  title       = var.lab_title
  description = "HQ with 2 edge routers, MPLS WAN (3x PE + 4x P), single-router ISP, two dual-homed branches (2 routers + L3 switch each). IOL only."
}
```

Nodes and links use ```for_each``` over the maps we just created:

```hcl
resource "cml2_node" "this" {
  for_each = local.nodes

  lab_id          = cml2_lab.this.id
  label           = each.key
  nodedefinition  = each.value.nodedefinition
  imagedefinition = each.value.image
  x               = each.value.x
  y               = each.value.y
  tags            = each.value.tags
  configuration   = file("${path.module}/configs/${each.key}.cfg")
}

resource "cml2_link" "this" {
  for_each = local.links

  lab_id = cml2_lab.this.id
  node_a = cml2_node.this[each.value.a].id
  slot_a = each.value.slot_a
  node_b = cml2_node.this[each.value.b].id
  slot_b = each.value.slot_b
}
```

Because the map keys are the node labels, the links can reference nodes by name, like ```cml2_node.this["PE1"]```, which keeps everything readable.


## Day-0 configurations

Every node boots with a startup configuration loaded from ```configs/<NODE>.cfg``` through the ```file()``` function, as you can see in the ```cml2_node``` resource above. CML injects that config before the first boot, so the lab comes up with addressing and routing already in place.

***What's in the day-0 configs:***
- P routers: IS-IS level-2 with wide metrics, per-prefix and remote LFA, LDP, MPLS TE extensions, RSVP on every core link, and automatic FRR backup tunnels
- PE routers: everything a P router has, plus VRF ```CUSTOMER```, iBGP VPNv4 full mesh between PE loopbacks, eBGP to the CEs under the VRF, and TE tunnels between PE1 and the other PEs (explicit paths with a dynamic fallback, autoroute, and fast reroute)
- MPLS-facing CEs: OSPFv2 inside the site, eBGP to the PE advertising the site /16 summary, and BGP redistributed into OSPF
- Internet-facing CEs: OSPFv2 inside the site, eBGP to the ISP receiving only a default route, and NAT overload on the Internet interface
- Overlay: GRE tunnels from each branch RTR2 to both HQ routers, running eBGP as the backup WAN. Local preference and OSPF metrics keep MPLS as the preferred path
- L3 switches: a transit VLAN toward both site routers, an SVI running OSPFv2, and a loopback simulating the users LAN
- ISP: eBGP to every Internet-facing CE, sending only a default route, and a loopback simulating an Internet destination

Here's a short example from P1, showing the core features enabled on each core link and the LFA configuration under IS-IS:

```
interface Ethernet0/2
 description CORE to P3 Ethernet0/0
 ip address 10.11.13.11 255.255.255.0
 ip router isis CORE
 isis network point-to-point
 mpls ip
 mpls traffic-eng tunnels
 ip rsvp bandwidth
 no shutdown
!
router isis CORE
 net 49.0001.0000.0000.0011.00
 is-type level-2-only
 metric-style wide
 passive-interface Loopback0
 mpls traffic-eng router-id Loopback0
 mpls traffic-eng level-2
 fast-reroute per-prefix level-2 all
 fast-reroute remote-lfa level-2 mpls-ldp
```

The full breakdown of every config, including the TE tunnels, NAT, and the failover knobs, is in the repository README.

> **Note:** CML only applies the day-0 config at the node's first boot. If you edit a file under ```configs/``` after the lab is running, Terraform will replace (destroy and recreate) that node to apply the new config.


## Starting the lab

At this point, Terraform would create the lab, the nodes, and the links, but nothing would be running. That's where the ```cml2_lifecycle``` resource comes in:

```hcl
resource "cml2_lifecycle" "this" {
  lab_id = cml2_lab.this.id
  state  = "STARTED"

  update_triggers = {
    for name, node in cml2_node.this : name => "${node.id}:${node.generation}"
  }

  depends_on = [cml2_node.this, cml2_link.this]
}
```

***Key points:***
- ```state = "STARTED"``` is the reason the lab starts automatically right after it's created. If you prefer to build the lab and start it yourself from the CML UI, use ```DEFINED_ON_CORE``` instead. ```STOPPED``` stops a running lab without deleting it.
- ```update_triggers``` makes sure the lifecycle resource notices when a node is replaced or its config changes, so replaced nodes get started again in the same apply.
- ```depends_on``` makes the lifecycle wait for every node and link, so the lab only starts once the full topology exists.


## Smart annotations

When you open the lab in CML, you'll notice colored shapes grouping the devices by site. Those are CML ***smart annotations***, and they're generated automatically from the node tags we set in the nodes map (```hq```, ```mpls```, ```internet```, ```branch1```, and ```branch2```). We get them for free just by tagging the nodes.

One limitation: smart annotations are always drawn as rounded shapes around the tagged nodes, and their shape can't be changed. If you'd rather have square boxes, the provider has a ```cml2_annotation``` resource that can draw rectangles and text, but you'd need to remove the tags to get rid of the smart annotations. I decided to keep them.


## Outputs

Finally, ```outputs.tf``` returns the lab ID, a direct link to the lab in the CML UI, and the ID of every node:

```hcl
output "lab_url" {
  description = "Direct link to the lab in the CML UI"
  value       = "${var.cml_address}/lab/${cml2_lab.this.id}"
}
```

------

# Deploying the lab

## Initializing the Terraform configuration

To initialize Terraform and download the CML2 provider:

```shell
terraform init
```

This also creates a ```.terraform.lock.hcl``` file, which pins the exact provider version. It's a good idea to commit it, so everyone using the repository gets the same provider.


## Planning the changes

To preview what Terraform is going to create:

```shell
terraform plan -out=cml_lab
```

You should see 42 resources to add: 1 lab, 17 nodes, 23 links, and the lifecycle resource.


## Applying the changes

Apply the planned changes:

```shell
terraform apply "cml_lab"
```

Terraform creates the lab, the nodes, and the links, and then the lifecycle resource starts everything. Once it finishes, Terraform displays the outputs, including the ```lab_url``` that takes you straight to the lab in the CML UI.

Here's how the lab looks in CML once everything is up and running:

<!-- TODO: save the CML screenshot as static/deploying-cml-labs-with-terraform/cml-topology.png -->
![Lab topology running in CML](/deploying-cml-labs-with-terraform/cml-topology.png)

Running ```terraform plan``` again right after should report no changes. That's a nice way to confirm what's running on CML matches what's in the code.


## Verifying the lab

Once all nodes are booted, open a console on any device and check the control plane. A few commands I like to start with:

```
show isis neighbors
show mpls ldp neighbor
show mpls traffic-eng topology brief
show bgp vpnv4 unicast all summary
show ip ospf neighbor
show ip route
```

And a few more for the day-N technologies:

```
show mpls traffic-eng tunnels brief
show isis fast-reroute summary
show ip nat translations
show ip bgp summary
```

To test the WAN failover, shut the PE-CE link on a branch (for example ```Ethernet0/1``` on BR1-RTR1) and run a traceroute from that branch switch to HQ. Traffic moves to the GRE overlay, and goes back to MPLS once the link is up again.


## Terraform destroy

Just like on any other Terraform project, tearing the whole lab down is a single command:

```shell
terraform destroy
```

Terraform stops the lab, wipes the nodes, and deletes it from CML. Running ```terraform apply``` again brings back exactly the same lab, with the same topology and the same configs.

------

# Gotchas

A few things I ran into while building this project:

- ***Config changes replace nodes:*** since day-0 configs are only applied at first boot, changing a config file recreates the node. Changing many configs at once on a running lab means a lot of nodes getting destroyed and recreated.
- ***Lifecycle errors on node replacement:*** with provider 0.10.1, any apply that replaces nodes on a running lab ends with ```Provider produced inconsistent result after apply``` on ```cml2_lifecycle```, complaining that an element of ```.nodes``` vanished or appeared. It's harmless. The nodes are replaced and started, and running ```terraform plan``` right after reports no changes.
- ***Interface slots:*** if you don't set ```slot_a``` and ```slot_b```, CML uses the next free interface, which may not match your configs. I recommend always setting them.
- ***Credentials:*** ```terraform.tfvars``` stores the password in plain text. Make sure it stays out of Git, and keep your state files local or on an encrypted backend.

------

# What's next

With the lab deployed as code, we have a solid base to work on. In the next posts, I'll use this topology to take a closer look at:

- RSVP and MPLS TE tunnels, with fast reroute
- LFA and remote LFA with IS-IS
- Internet access and WAN failover over the GRE overlay

------

# References

- [CiscoDevNet/cml2 Terraform provider](https://registry.terraform.io/providers/CiscoDevNet/cml2/latest/docs)
- [Cisco Modeling Labs documentation](https://developer.cisco.com/docs/modeling-labs/)
- [cisco-cml-iac repository](https://github.com/lyraguilherme/cisco-cml-iac)
