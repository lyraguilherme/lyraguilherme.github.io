---
title: "Fortigate VPN Parser"
date: 2025-07-19T17:46:11-03:00
draft: false
cover:
    image: /fortigate-vpn-parser/fg-vpn-cover.png
    alt: 'Fortigate VPN Parser'
    caption: 'Fortigate VPN Parser'
---

When you're handed a Fortigate backup config and need to document dozens of IPsec VPNs — but have no access to the device — manual parsing becomes a pain.

To solve that, I built a Python script that automates the process.

# No Device Access Needed 🥷

This script works entirely offline. All it needs is a raw FortiGate config file (e.g., from show full-config) — no API or CLI access required.

# AI-Assisted Parsing 🤖

Some of the trickiest parts were handling nested edit/set/next blocks and mapping address groups to actual subnets. AI helped speed up the development and sanity-check the parsing logic.

# What It Does 🔍

- Parses phase1-interface and phase2-interface IPsec VPN definitions
- Resolves source/destination address groups
- Maps all referenced address objects to subnets
- Displays the result in a clean, console-based table
- Exports structured output as a dictionary to a .txt file

# Output Example 📦

The output clearly shows which local/remote subnets are used in each VPN — perfect for audits, migrations, or documentation.

![Fortigate VPN Parser](/fortigate-vpn-parser/fg-vpn-parser.png)

# Using the script

The full code is available on my GitHub:
https://github.com/lyraguilherme/networkscripts/tree/main/fortigate_vpn_parser


## Installing requirements

- Make sure you're running Python 3.7+
- Install Rich library
```bash
pip install rich
```

## Running the script
- Save the full Fortigate config in the same folder as the script
- Run the script using the example syntax below

**Usage:**
```bash
python fg_vpn_parser.py <path_to_config.txt>
```