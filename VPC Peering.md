# VPC Peering

**VPC Peering** is a networking connection between two Virtual Private Clouds (VPCs) that allows them to communicate with each other **as if they are in the same network**.

---

## 🔹 Simple Idea

Think like this:

👉 You have **VPC A** and **VPC B**
👉 Normally, they are isolated (no communication)
👉 With **VPC Peering**, you create a private connection between them

So now:

* EC2 in VPC A can talk to EC2 in VPC B
* Traffic stays **inside AWS network (secure + no internet)**



## 🔹 Key Points 

### ✅ 1. Private Communication

* Uses **private IP addresses**
* No need for Internet Gateway, NAT, or VPN

---

### ✅ 2. Bidirectional (Two-way)

* Both VPCs can send and receive traffic

---

### ✅ 3. Route Tables Required

* You must manually update route tables:

  ```
  VPC A route → VPC B CIDR
  VPC B route → VPC A CIDR
  ```

---

### ✅ 4. No Overlapping CIDR

❌ Not allowed:

```
VPC A → 10.0.0.0/16  
VPC B → 10.0.0.0/16  ❌
```

✔ Must be different:

```
VPC A → 10.0.0.0/16  
VPC B → 192.168.0.0/16 ✔
```

---

### ✅ 5. Works Across Regions & Accounts

* Same region ✔
* Different region ✔
* Different AWS accounts ✔

---

### ❌ 6. No Transitive Peering (VERY IMPORTANT)

If:

```
VPC A ↔ VPC B  
VPC B ↔ VPC C  
```

👉 Then:

```
VPC A ❌ cannot talk to VPC C
```

---

## 🔹 Real Example

* You have:

  * **Frontend app** in VPC A
  * **Database (RDS)** in VPC B

👉 Use VPC peering so app can securely access DB.

---

## 🔹 When to Use

Use VPC Peering when:

* You want **simple, direct connection**
* Few VPCs (small architecture)
* Low latency communication

---

## 🔹 When NOT to Use

Avoid when:

* Many VPCs (complex network)
* Need centralized routing

👉 Instead use:

* AWS Transit Gateway

---

Got it — I’ll walk you **click by click** so you won’t get confused 👍

---

# 🪜 How to Create VPC (STEP 1 & STEP 2)

## 🔹 Open AWS VPC Dashboard

1. Login to **Amazon Web Services Console**
2. In search bar → type **VPC**
3. Click **VPC (Virtual Private Cloud)**

---

### Follow these steps:

1. Click **Create VPC**

2. Select:
   👉 **VPC only** (Important — don’t choose auto setup)

3. Fill details:

* Name tag: `VPC-A`
* IPv4 CIDR block: `10.0.0.0/16`
* IPv6: **No IPv6** (leave default)

4. Click **Create VPC**

✅ Done — your first VPC is ready

---

## 🔹 Create VPC-B

👉 Repeat same steps:

1. Click **Create VPC**
2. Choose **VPC only**

Fill:

* Name tag: `VPC-B`
* IPv4 CIDR block: `192.168.0.0/16`

3. Click **Create**

✅ Done — second VPC ready

---

## 🎯 What You Should See

After creation:

* 2 VPCs in list:

  * `VPC-A` → 10.0.0.0/16
  * `VPC-B` → 192.168.0.0/16

---

## ⚠️ Important Beginner Tips

* CIDR must be **different** (you did correct 👍)
* Always choose **"VPC only"**
* Don’t worry about subnets now — next step we create it

---

## 👍 Quick Check

If you want to confirm:

👉 Click VPC-A → you should see:

* CIDR: `10.0.0.0/16`

👉 Click VPC-B → you should see:

* CIDR: `192.168.0.0/16`

---
Nice — you’re moving correctly 👍
I’ll guide you **exact clicks + what to select** so you don’t get stuck.

---

# 🪜 STEP 3: Create Subnets (Detailed)

## 🔹 Open Subnets Page

1. Go to **Amazon Web Services Console**
2. Search → **VPC**
3. Left side → Click **Subnets**
4. Click **Create subnet**

---

Fill like this:

* VPC ID → select **VPC-A**
* Subnet name → `Subnet-A`
* Availability Zone → choose any (example: ap-south-1a)
* IPv4 CIDR → `10.0.1.0/24`

👉 Click **Create subnet**

---

## 🔹 Create Subnet-B (for VPC-B)

Repeat same:

* VPC ID → **VPC-B**
* Subnet name → `Subnet-B`
* CIDR → `192.168.1.0/24`

👉 Click **Create subnet**

---

# 🪜 STEP 4: Create Internet Gateway (IGW)


1. Left menu → **Internet Gateways**
2. Click **Create internet gateway**

Fill:

* Name → `IGW-A`

👉 Click **Create**

---

## 🔹 Attach IGW-A to VPC-A

1. Select **IGW-A**
2. Click **Actions → Attach to VPC**
3. Choose **VPC-A**
4. Click **Attach**

---

## 🔹 Repeat for VPC-B

* Create → `IGW-B`
* Attach → **VPC-B**

1. Left menu → **Route Tables**
2. Click **Create route table**

Fill:

* Name → `RT-A`
* VPC → **VPC-A**

👉 Click **Create**

---

## 🔹 Add Internet Route (Important)

1. Select **RT-A**
2. Go to **Routes tab**
3. Click **Edit routes → Add route**

Fill:

```
Destination: 0.0.0.0/0
Target: IGW-A
```

👉 Save

---

## 🔹 Associate Subnet-A

1. Go to **Subnet associations tab**
2. Click **Edit associations**
3. Select **Subnet-A**
4. Save

---

## 🔹 Repeat for VPC-B

Create:

* Route Table → `RT-B`
* Attach → VPC-B

Add route:

```
0.0.0.0/0 → IGW-B
```

Associate:

* Subnet-B

---

# 🎯 What You Completed

✅ Subnet created in both VPCs
✅ Internet access enabled
✅ Route tables configured
✅ Subnets linked properly

---

# 🪜 STEP 6: Launch EC2 + Connect + Install Apache

We will:

* Launch **2 EC2 instances** (one in each VPC)
* Connect using SSH
* Install Apache web server

---

# 🔹 Part 1: Launch EC2 in VPC-A

## 📍 Go to EC2

1. Open **Amazon Web Services Console**
2. Search → **EC2**
3. Click **Launch Instance**

---

## ⚙️ Configure EC2-A

Fill like this:

### 🏷️ Basic Details

* Name → `EC2-A`

### 🖥️ AMI

* Select: **Amazon Linux 2** (or Amazon Linux 2023)

### 💻 Instance Type

* `t2.micro` (free tier)

### 🔑 Key Pair

* Click **Create new key pair**
* Name: `my-key`
* Download `.pem` file ⚠️ (important for SSH)

---

### 🌐 Network Settings (VERY IMPORTANT)

Click **Edit**

* VPC → **VPC-A**
* Subnet → **Subnet-A**
* Auto-assign Public IP → **Enable**

---

### 🔐 Security Group

Create new:

Allow:

* SSH → My IP
* HTTP → Anywhere (0.0.0.0/0)
* (Optional) ICMP → Anywhere

---

👉 Click **Launch Instance**

---

# 🔹 Part 2: Launch EC2 in VPC-B

Repeat same steps:

* Name → `EC2-B`
* VPC → **VPC-B**
* Subnet → **Subnet-B**
* Use same key pair (`my-key`)

---

# 🔹 Part 3: Connect to EC2 (SSH)

## 💻 Get Public IP

* Go to EC2 → Instances
* Copy **Public IP** of EC2-A

---

## 🧑‍💻 Connect from terminal

```bash
chmod 400 my-key.pem
ssh -i my-key.pem ec2-user@<PUBLIC-IP>
```

---

# 🔹 Part 4: Install Apache (Web Server)

## 📦 Run this inside EC2

```bash
sudo yum update -y
sudo yum install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
```

---

## 🔹 Add Test Page

### 👉 In EC2-A

```bash
echo "Hello from VPC-A" | sudo tee /var/www/html/index.html
```

### 👉 In EC2-B

```bash
echo "Hello from VPC-B" | sudo tee /var/www/html/index.html
```

---

# 🔹 Part 5: Test in Browser

👉 Open browser:

```
http://<PUBLIC-IP-OF-EC2-A>
```

👉 You should see:

```
Hello from VPC-A
```

---

# 🎯 What You Completed

✅ EC2 launched in both VPCs
✅ SSH connection working
✅ Apache installed
✅ Web page running

---
Great — this is the **final and most important step** 🔥
Now we will connect both VPCs and test using **private IP (real DevOps concept)**

---

# 🪜 STEP 7: VPC Peering + Routing + Testing

---

# 🔹 Part 1: Create VPC Peering

## 📍 Go to Peering Connections

1. Open **Amazon Web Services Console**
2. Search → **VPC**
3. Left menu → **Peering Connections**
4. Click **Create peering connection**

---

* Name → `A-to-B`
* VPC (Requester) → **VPC-A**
* Account → My account
* Region → Same region
* VPC (Accepter) → **VPC-B**

👉 Click **Create**

---

## 🔹 Accept Peering

1. Select the connection
2. Click **Actions → Accept request**
3. Status should become:
   ✅ **Active**

---

# 🔹 Part 2: Update Route Tables ⚠️ (MOST IMPORTANT)

👉 Without this, it **won’t work**

---

## 🔸 Update VPC-A Route Table (RT-A)

1. Go → **Route Tables**
2. Select **RT-A**
3. Go → **Routes → Edit routes → Add route**

Add:

```id="azb5bf"
Destination: 192.168.0.0/16
Target: Peering Connection (A-to-B)
```

👉 Save

---

## 🔸 Update VPC-B Route Table (RT-B)

Add:

```id="0yu22o"
Destination: 10.0.0.0/16
Target: Peering Connection (A-to-B)
```

👉 Save

---

# 🔹 Part 3: Update Security Groups

👉 This allows traffic between VPCs

---

## 🔸 EC2-A Security Group

Add inbound rule:

```id="p5v1ew"
Type: All traffic (or ICMP + HTTP)
Source: 192.168.0.0/16
```

---

## 🔸 EC2-B Security Group

Add:

```id="bwdx38"
Source: 10.0.0.0/16
```

---

# 🔹 Part 4: Get Private IPs

👉 Go to EC2 → Instances

* EC2-A → copy **Private IP** (example: `10.0.1.10`)
* EC2-B → copy **Private IP** (example: `192.168.1.10`)

---

# 🔹 Part 5: Test Connectivity ✅

## 🔸 From EC2-A → EC2-B

SSH into EC2-A:

```bash id="4mvrwo"
ping <PRIVATE-IP-OF-EC2-B>
```

👉 If working:

```
64 bytes from ...
```

---

## 🔸 Test Web (IMPORTANT)

```bash id="8yu4av"
curl http://<PRIVATE-IP-OF-EC2-B>
```

👉 Output:

```
Hello from VPC-B
```

---

# 🎯 Final Result

✅ VPC Peering working
✅ EC2 to EC2 communication
✅ Using **private IP only**
✅ No internet involved

---

# ⚠️ If It Doesn’t Work (Debug Checklist)

Check these 4 things:

1. ✅ Peering status = Active
2. ✅ Route tables updated both sides
3. ✅ Security groups allow CIDR
4. ✅ Using **private IP (not public)**

---

# 🚀 You Completed a Real DevOps Project 🎉







