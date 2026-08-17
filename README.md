# 🏢 Enterprise Multi-Site Network Architecture (IaC & VLSM)

## 📌 Project Overview
This project models and provisions an Enterprise-grade multi-site network architecture connecting a Headquarter (HQ) site and a Data Center (DC). It bridges traditional on-premises network engineering (simulated in Cisco Packet Tracer) with Cloud Infrastructure as Code using **AWS VPCs, Transit Gateway, and Terraform**.

---

## 📐 Zero-IP-Waste Subnetting Plan (VLSM)

The network is carved out from the base block `172.16.0.0/24` to eliminate IP address waste across two distinct sites connected via a `/30` WAN Point-to-Point link:

| Department / Site | VLAN | Subnet Type | CIDR Block | Subnet Mask | Usable Host Range |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Sales & Marketing (HQ)** | VLAN 10 | Internal User | `172.16.0.0/25` | `255.255.255.128` | `172.16.0.1` - `172.16.0.126` |
| **Devs & Apps (HQ)** | VLAN 20 | Internal User | `172.16.0.128/26` | `255.255.255.192` | `172.16.0.129` - `172.16.0.190` |
| **Admin & Finance (DC)** | VLAN 30 | Internal Mgmt | `172.16.0.192/27` | `255.255.255.224` | `172.16.0.193` - `172.16.0.222` |
| **Database Servers (DC)** | VLAN 40 | Isolated Data | `172.16.0.224/28` | `255.255.255.240` | `172.16.0.225` - `172.16.0.238` |
| **WAN Point-to-Point** | N/A | Inter-Site Link | `172.16.0.240/30` | `255.255.255.252` | `172.16.0.241` - `172.16.0.242` |

---

## ☁️ Cloud Mapping (AWS Architecture)
* **HQ VPC & DC VPC:** Represented as two isolated Virtual Private Clouds in AWS.
* **WAN Link (`/30`):** Modeled using **AWS Transit Gateway (TGW)** for scalable cross-VPC routing.
* **Inter-VLAN Routing:** Translated into subnets paired with explicit **AWS Route Tables**.

---

## 🛠️ Infrastructure as Code (Terraform Snippet)

```hcl
# AWS Transit Gateway for Inter-VPC Routing
resource "aws_ec2_transit_gateway" "tgw" {
  description = "Enterprise WAN Transit Gateway connecting HQ and DC"
  tags = {
    Name = "Enterprise-TGW"
  }
}

# Subnet Allocation for Sales (/25)
resource "aws_subnet" "sales_subnet" {
  vpc_id            = aws_vpc.hq_vpc.id
  cidr_block        = "172.16.0.0/25"
  availability_zone = "us-east-1a"

  tags = {
    Name = "HQ-Sales-Subnet"
  }
}
