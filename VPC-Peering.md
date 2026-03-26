## VPC Peering

**VPC Peering** is a networking connection between two Virtual Private Clouds (VPCs) in Amazon Web Services that allows them to communicate with each other as if they are in the same network.

---

## Simple Idea

Think like this:

You have **VPC A** and **VPC B**
Normally, they are isolated (no communication)
With VPC Peering, you create a private connection between them

So now:

* EC2 in VPC A can talk to EC2 in VPC B
* Traffic stays inside AWS network (secure and no internet)

---

## Key Points

### 1. Private Communication

* Uses private IP addresses
* No need for Internet Gateway, NAT, or VPN

---

### 2. Bidirectional (Two-way)

* Both VPCs can send and receive traffic

---

### 3. Route Tables Required

You must manually update route tables:

```
VPC A route → VPC B CIDR
VPC B route → VPC A CIDR
```

---

### 4. No Overlapping CIDR

Not allowed:

```
VPC A → 10.0.0.0/16  
VPC B → 10.0.0.0/16
```

Must be different:

```
VPC A → 10.0.0.0/16  
VPC B → 192.168.0.0/16
```

---

### 5. Works Across Regions & Accounts

* Same region
* Different region
* Different AWS accounts

---

### 6. No Transitive Peering (Very Important)

If:

```
VPC A ↔ VPC B  
VPC B ↔ VPC C  
```

Then:

```
VPC A cannot talk to VPC C
```

---

## Real Example

* Frontend app in VPC A
* Database (RDS) in VPC B

Use VPC peering so the application can securely access the database.

---

## When to Use

Use VPC Peering when:

* You want a simple, direct connection
* Few VPCs (small architecture)
* Low latency communication

---

## When Not to Use

Avoid when:

* Many VPCs (complex network)
* Need centralized routing

Instead use:

* AWS Transit Gateway

---

# Step 1 & 2: Create VPCs

## Open VPC Dashboard

1. Login to AWS Console
2. Search for VPC
3. Click VPC

---

### Create VPC-A

1. Click Create VPC
2. Select VPC only

Fill:

* Name: `VPC-A`
* CIDR: `10.0.0.0/16`
* IPv6: No IPv6

Click Create VPC

---

### Create VPC-B

Repeat:

* Name: `VPC-B`
* CIDR: `192.168.0.0/16`

---

## Expected Result

* VPC-A → 10.0.0.0/16
* VPC-B → 192.168.0.0/16

---

# Step 3: Create Subnets

## Subnet-A

* VPC: VPC-A
* Name: `Subnet-A`
* AZ: ap-south-1a
* CIDR: `10.0.1.0/24`

---

## Subnet-B

* VPC: VPC-B
* Name: `Subnet-B`
* CIDR: `192.168.1.0/24`

---

# Step 4: Internet Gateway

## Create and Attach

* Create `IGW-A` → attach to VPC-A
* Create `IGW-B` → attach to VPC-B

---

# Step 5: Route Tables

## VPC-A

* Create `RT-A`
* Add route:

```
0.0.0.0/0 → IGW-A
```

* Associate with Subnet-A

---

## VPC-B

* Create `RT-B`
* Add route:

```
0.0.0.0/0 → IGW-B
```

* Associate with Subnet-B

---

# Step 6: Launch EC2 Instances

## EC2-A

* AMI: Amazon Linux
* Type: t2.micro
* VPC: VPC-A
* Subnet: Subnet-A
* Public IP: Enabled

Security Group:

* SSH → My IP
* HTTP → Anywhere

---

## EC2-B

Same setup but:

* VPC: VPC-B
* Subnet: Subnet-B

---

## SSH Connection

```bash
chmod 400 my-key.pem
ssh -i my-key.pem ec2-user@<PUBLIC-IP>
```

---

## Install Apache

```bash
sudo yum update -y
sudo yum install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
```

---

## Add Test Page

EC2-A:

```bash
echo "Hello from VPC-A" | sudo tee /var/www/html/index.html
```

EC2-B:

```bash
echo "Hello from VPC-B" | sudo tee /var/www/html/index.html
```

---

## Test in Browser

```
http://<PUBLIC-IP>
```

---

# Step 7: VPC Peering + Routing + Testing

## Create Peering

* Name: `A-to-B`
* Requester: VPC-A
* Accepter: VPC-B

Accept the request → Status should be Active

---

## Update Route Tables

### VPC-A (RT-A)

```
Destination: 192.168.0.0/16
Target: Peering Connection
```

---

### VPC-B (RT-B)

```
Destination: 10.0.0.0/16
Target: Peering Connection
```

---

## Update Security Groups

### EC2-A

```
Allow: 192.168.0.0/16
```

### EC2-B

```
Allow: 10.0.0.0/16
```

---

## Get Private IPs

* EC2-A → example: 10.0.1.10
* EC2-B → example: 192.168.1.10

---

## Test Connectivity

### Ping

```bash
ping <PRIVATE-IP>
```

---

### Curl Test

```bash
curl http://<PRIVATE-IP>
```

Expected:

```
Hello from VPC-B
```

---

# Final Result

* VPC Peering working
* EC2 to EC2 communication
* Using private IP only
* No internet involved

---

# Debug Checklist

1. Peering status = Active
2. Route tables updated on both sides
3. Security groups allow CIDR
4. Using private IP (not public)
