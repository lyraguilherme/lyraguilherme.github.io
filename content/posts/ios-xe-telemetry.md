---
title: 'Cisco IOS XE Model-Driven Telemetry'
date: 2023-02-02T14:07:09-03:00
draft: false
cover:
    image: /static/static/images/ios-xe-telemetry/mdt_dashboard.jpeg
    alt: 'Model-Driven Telemetry'
    caption: 'Cisco IOS XE Model-Driven Telemetry'
---

In this post we're going to explore some Cisco IOS XE capabilities such as **Streaming Telemetry** and **Guestshell**.

There is a ton of content available on **Cisco DevNet** explaining **Model-Driven Telemetry** theory in great detail, so I highly suggest you take some time to browse through the links I've listed under the **Reference** of this post.

## Summary ##

* My lab router is a Cisco ISR C1111-4G running IOS XE 17.6.3a. The same environment can be set up on a CSR1000v or Catalyst 8000v running on Cisco CML for example.

* You will notice I'm using subnet 192.168.111.0/24 on an interface named VirtualPortGroup0. This is a virtual interface that will be used for communication between the Guestshell and IOS XE. Feel free to change the IP addressing as needed.

* I needed to NAT overload the above subnet out the router's WAN interface so that Guestshell could to reach the Internet. This is optional and you should adjust as needed, depending on your environment.

* We will be installing packages on Guestshell's OS using the Yum package manager. For this reason, configuring Guestshell to query a real DNS server is mandatory. I used Google's 8.8.8.8 for this lab.

> **Note:** Guestshell has limited resources so this kind of setup should be used for testing or labbing purposes only. If you plan on collecting MDT on a production environment, make sure to size your servers accordingly.

## Guestshell ##

First of all, I want to make sure you understand the implications of running a database under Guestshell environment. Guestshell is a Linux container running on top of IOS XE, so you will have limited resources. Depending on the platform you're running, you should be able to increase resources for Guestshell, though I'm not going to cover that in this post.

To keep it short, if you plan on streaming **Model-Driven Telemetry** on a production environment, please make sure you host your Collector and Databases on dedicated servers and configure data retention accordingly.

## Enabling IOX service ##

Ok, our first step is to enable the IOX service under IOS XE.

```
conf t
iox
end
```

## Verifying IOX status ##

If you just enabled IOX, give it a few minutes and check the services status with the following command:

```
Router#show iox

IOx Infrastructure Summary:
---------------------------
IOx service (CAF)              : Running
IOx service (HA)               : Not Supported 
IOx service (IOxman)           : Running 
IOx service (Sec storage)      : Not Running 
Libvirtd 5.5.0                 : Running
```

> **Note:** Make sure services CAF, IOxman and Libvirtd are running before proceding to the next steps.

## Setting App-Hosting Parameters ##

Another step before enabling Guestshell is to set the networking configuration the container will use. In my lab I will be using the following settings:
* **Guestshell IP:** 192.168.111.2/24
* **Default Gateway:** 192.168.111.1
* **DNS server:** 8.8.8.8

Feel free to change the IP addressing as needed, just make sure to use a valid DNS server.

```
conf t
!
app-hosting appid guestshell
 app-vnic gateway0 virtualportgroup 0 guest-interface 0
  guest-ipaddress 192.168.111.2 netmask 255.255.255.0
 app-default-gateway 192.168.111.1 guest-interface 0
 name-server0 8.8.8.8
 !
end
```

## NAT configuration - optional ##

My upstream router (ISP) only supports static routing and, since this is just a lab, I was easier for me to just NAT overload all traffic from the Guestshell towards my LAN instead of creating static routes on my ISP's router. Again, this is an optional step. Just keep in mind that Guestshell will need to reach the Internet.

In case you do need to NAT, here's the configuration I'm using:

```
conf t
!
ip access-list extended NAT_ACL
 10 permit ip 192.168.111.0 0.0.0.255 any
!
ip nat inside source list NAT_ACL interface GigabitEthernet0/0/0 overload
!
interface GigabitEthernet0/0/0
 description This is my routers WAN interface 
 ip nat outside
!
interface VirtualPortGroup0
 description This virtual interface will be Guestshells default gateway
 ip address 192.168.111.1 255.255.255.0
 ip nat inside
!
end
```

## Enabling Guestshell ##

After enabling **IOX service** and configuring App-Hosting parameters, the next step is to instantiate the Guestshell container:

```
Router#guestshell enable
Interface will be selected if configured in app-hosting
Please wait for completion

Guestshell enabled successfully
```

Guestshell is now up and running, so we can get access to the Linux shell:

```
Router#guestshell run bash
[guestshell@guestshell ~]$ 
```

## Setting up the Collector and Database ##

At this point, we need to install and configure our Collector and our Time Series Database. For this lab I'm using Telegraf and InfluxDB, but there are other options out there that will achieve the same results.

From the Guestshell Bash:

```
cat <<EOF | sudo tee /etc/yum.repos.d/influxdb.repo
[influxdb]
name = InfluxDB Repository - RHEL \$releasever
baseurl = https://repos.influxdata.com/rhel/\$releasever/\$basearch/stable
enabled = 1
gpgcheck = 1
gpgkey = https://repos.influxdata.com/influxdata-archive_compat.key
EOF
```

Install **Telegraf** and **InfluxDB** using Yum:

```
sudo yum install telegraf influxdb2
```

Now let's edit Telegraf's config file. Edit **/etc/telegraf/telegraf.conf** and uncomment the following lines:

```
[[outputs.influxdb_v2]]
  urls = ["http://127.0.0.1:8086"]
  # leave the token empty for now
  token = ""
  organization = "MDT-LAB"
  bucket = "telegraf"
 
[[inputs.cisco_telemetry_mdt]]
  transport = "grpc"
  service_address = ":57000"
  max_msg_size = 4000000
```

Save the file and start the services:

```
sudo systemctl start influxdb
sudo systemctl start telegraf
```

Now that **InfluxDB** is running, let's use the browser to open the GUI at **http://<your_guestshell_ip>:8086**

![InfluxDB Get Started](/static/images/ios-xe-telemetry/image_1.png)
*Click GET STARTED and (1) set up an account, (2) create a bucket and (3) create an API Token for Telegraf*

![InfluxDB Initial User Setup](/static/images/ios-xe-telemetry/image_2.png)
*Click CONFIGURE LATER*

![InfluxDB Getting Started](/static/images/ios-xe-telemetry/image_3.png)
*Expand the left-side menu and click LOAD DATA*

![InfluxDB Load Data](/static/images/ios-xe-telemetry/image_4.png)
*Go to API TOKENS and click GENERATE API TOKEN*

![InfluxDB Custom API Token](/static/images/ios-xe-telemetry/image_5.png)
*Give it read+write permissions under bucket named telegraf and click GENERATE*

![InfluxDB Telegraf API Token](/static/images/ios-xe-telemetry/image_6.png)
*API Token will be used on Telegraf config*

Now, we need to go back and edit **/etc/telegraf/telegraf.conf** once again to include the API token:

```
# Token for authentication
token = "insert_your_token_here"
```

Save the file and restart Telegraf service:

```
sudo systemctl restart telegraf
```

At this point our collector and database are ready to go!

## Model-Driven Telemetry ##

As Cisco states: *Model-driven Telemetry (MDT) provides a mechanism to stream data from an MDT-capable device to a destination. The data to be streamed is subscribed from a data set in a YANG model. The data from the subscribed data set is streamed out to the destination either at a configured periodic interval or only when an event occurs.*

On the next steps we're going to enable **NETCONF** on IOS XE and also configure our Telemetry Subscriptions that we want to stream to the collector (Telegraf running on Guestshell).

## NETCONF ##

Let's enable NETCONF:

```
conf t
netconf-yang
end
```

## Telemetry Subscriptions ##

Still on IOS XE, the next step is to configure the **Telemetry Subscriptions**. By looking at the configuration below, you will notice we will be using gRPC as the transport protocol.

The key here is that **YANG XPaths** are used to select the information we want to stream. If you want to explore further, I suggest using **Cisco YANG Suite** (see link at the end of the post).

Creating **Telemetry Subscriptions** on IOS XE:

```
telemetry ietf subscription 100
 encoding encode-kvgpb
 filter xpath /process-cpu-ios-xe-oper:cpu-usage/cpu-utilization/five-seconds
 source-address 192.168.111.1
 stream yang-push
 update-policy periodic 500
 receiver ip address 192.168.111.2 57000 protocol grpc-tcp
 
telemetry ietf subscription 101
 encoding encode-kvgpb
 filter xpath /memory-ios-xe-oper:memory-statistics/memory-statistic
 source-address 192.168.111.1
 stream yang-push
 update-policy periodic 500
 receiver ip address 192.168.111.2 57000 protocol grpc-tcp

telemetry ietf subscription 200
 encoding encode-kvgpb
 filter xpath /interfaces-ios-xe-oper:interfaces/interface[name='GigabitEthernet0/0/0']/statistics
 source-address 192.168.111.1
 stream yang-push
 update-policy periodic 500
 receiver ip address 192.168.111.2 57000 protocol grpc-tcp

telemetry ietf subscription 201
 encoding encode-kvgpb
 filter xpath /interfaces-ios-xe-oper:interfaces/interface[name='GigabitEthernet0/1/1']/statistics
 source-address 192.168.111.1
 stream yang-push
 update-policy periodic 500
 receiver ip address 192.168.111.2 57000 protocol grpc-tcp

telemetry ietf subscription 300
 encoding encode-kvgpb
 filter xpath /interfaces/interface[name='GigabitEthernet0/0/0']/diffserv-info/diffserv-target-classifier-stats
 source-address 192.168.111.1
 stream yang-push
 update-policy periodic 500
 receiver ip address 192.168.111.2 57000 protocol grpc-tcp
 ```

The subscription IDs are arbitrary and locally significant only.

Most of the **XPaths** above are self-explanatory and you will probably notice that we will be monitoring the router's CPU, Memory and Statistics from interfaces **GigabitEthernet0/0/0** and **GigabitEthernet0/1/1**. Again, adjust the interfaces naming to match your environment.

If you saw my **QoS monitoring** post on LinkedIn, then take a closer look at the Subscription id 300. Note that /interfaces/interface[name='GigabitEthernet0/0/0']/diffserv-info/diffserv-target-classifier-stats is the **XPath** related to diffserv classifier statistics for my router's WAN interface.

## Collecting and Storing Telemetry Data ##

After configuring the **Telemetry Subscriptions** on IOS XE, let's go back to the **InfluxDB** dashboard and check if data is being received:

![InfluxDB Data Explorer](/static/images/ios-xe-telemetry/image_10.png)
*Check if MDT streams are being received*

Notice that **Telegraf** is now receiving **Telemetry Streams** from the router. From this point on, you can create a dashboard on **Telegraf** or even use **Grafana** to monitor everything you want.

So, going back to my **QoS monitoring** post, this is the configuration I used to create those nice graphs:

![InfluxDB Data Explorer](/static/images/ios-xe-telemetry/image_11.jpg)
*Selecting the desired data*

1. On the 1st filter, search for **diffserv**
2. On the 2nd filter, search for **rate**
3. On the 3rd filter there's no need to select anything, unless you want to create a graph with specific traffic classes only (notice on the image above that I didn't select anything there)
4. On the 4rd filter: select traffic direction (qos-inbound or qos-outbound)
5. Click **submit** and you'll get a live preview of the graph

If you want to add this specific graph to a dashboard, just click save as (top right of the screen) and create a new dashboard with your new graph:

![InfluxDB Data Explorer](/static/images/ios-xe-telemetry/image_12.jpg)
*Adding the graph to a new dashboard*

![InfluxDB Data Explorer](/static/images/ios-xe-telemetry/image_13.jpg)
*Some of the available options*

![InfluxDB MDT Dashboard](/static/images/ios-xe-telemetry/image_14.png)
*Dashboard example with inbound and outbound QoS traffic statistics*

## Conclusion ##

We configured a **Telegraf** collector and **InfluxDB** running over a **Guestshell** container. We also configured IOS XE to stream **Model-Driven Telemetry** using **gRPC** to the collector+database. 

## References ##
* Enterprise Streaming Telemetry and You: Getting Started with Model Driven Telemetry: https://blogs.cisco.com/developer/getting-started-with-model-driven-telemetry?ref=guilhermelyra.com

* Streaming Telemetry Quick Start Guide: https://developer.cisco.com/docs/ios-xe/?ref=guilhermelyra.com#!streaming-telemetry-quick-start-guide

* Cisco YANG Suite: https://developer.cisco.com/yangsuite/?ref=guilhermelyra.com

