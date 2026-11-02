# VPC Peering: Connecting Two AWS VPCs Privately

## Problem
Two separate networks (VPCs) need to talk to each other using private IP addresses, without sending traffic over the internet.

## Solution
Use **VPC peering**: a private connection between two VPCs. Then add routes in both VPCs so each side knows how to reach the other.

## Architecture

<img width="1657" height="1011" alt="architecture" src="https://github.com/user-attachments/assets/b81cf4c9-6e1a-4a60-8cb8-8485f37478d3" />


| Item | VPC-A | VPC-B |
|---|---|---|
| Name | Default VPC | Vpc-B-vpc |
| CIDR | 172.31.0.0/16 | 10.0.0.0/16 |
| Server | server-a (used to log in) | server-b (private IP 10.0.9.132) |
| Route to the other VPC | 10.0.0.0/16 to peering connection | 172.31.0.0/16 to peering connection |

The two CIDR ranges do **not overlap**, which is required for peering.

## Steps
1. Created a second VPC (`Vpc-B-vpc`, CIDR `10.0.0.0/16`) with one public subnet
2. Launched one EC2 instance in each VPC (`server-a` and `server-b`)
3. Created a peering connection from the default VPC to `Vpc-B-vpc` and accepted it (status: Active)
4. Added a route in the VPC-B route table: `172.31.0.0/16` to the peering connection
5. Added a route in the default VPC route table: `10.0.0.0/16` to the peering connection
6. Added an inbound rule in server-b's security group: All ICMP - IPv4 from `172.31.0.0/16`
7. Logged in to server-a and ran `ping 10.0.9.132`

## Testing

Peering connection is Active:

<img width="1365" height="767" alt="Screenshot 2026-11-02 123629" src="https://github.com/user-attachments/assets/61ab630f-4dac-4b99-8777-3012ca47208b" />


Route table of VPC-B:

<img width="1365" height="765" alt="Screenshot 2026-11-02 121521" src="https://github.com/user-attachments/assets/21aed583-5d72-46ce-963d-0d3e67ffa54c" />


Route table of the default VPC:

<img width="1365" height="767" alt="Screenshot 2026-11-02 121537" src="https://github.com/user-attachments/assets/2d3ea09d-3520-43c4-9e1e-bae1fbad4ea7" />


Ping from server-a to server-b succeeds (private connectivity works):

<img width="1365" height="767" alt="Screenshot 2026-11-02 123629" src="https://github.com/user-attachments/assets/fd50310f-3423-456e-bc56-60efde1855c6" />


After removing the peering route from the default VPC route table, the ping fails (100% packet loss):

<img width="887" height="767" alt="Screenshot 2026-11-02 123754" src="https://github.com/user-attachments/assets/239c42ce-be5f-481a-a483-268ac270660f" />


I added the route back and the ping worked again.

## Problems I faced and how I fixed them

| Problem | Cause | Fix |
|---|---|---|
| Error "A valid CIDR block has to be specified" | I typed a server IP (`10.0.9.16`) as the route destination | A route destination must be a network range, so I used `10.0.0.0/16` |
| I accidentally edited the internet route | I changed an existing row instead of adding a new one | Cancelled without saving, then used **Add route** |
| Ping showed 100% packet loss | I was pinging the wrong IP (`10.0.9.16`) | Copied the real private IP from the EC2 console (`10.0.9.132`) |

## Troubleshooting order (if ping fails)
1. Is the destination IP correct?
2. Is the peering connection Active?
3. Is there a route in **both** route tables, in the table linked to each server's subnet?
4. Does the destination security group allow ICMP from the other VPC's range?

## What I learned
- VPC peering connects two VPCs privately, and the CIDR ranges must not overlap
- Peering is not enough: each VPC needs a route to the other
- A route destination is a network range, not a single IP
- Security groups must allow the traffic (ICMP for ping)
- How to troubleshoot connectivity step by step

## Tools
AWS (VPC, VPC Peering, Route Tables, EC2, Security Groups), Linux
