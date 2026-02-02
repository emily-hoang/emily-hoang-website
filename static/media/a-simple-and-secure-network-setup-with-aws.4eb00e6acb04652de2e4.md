# A Simple & Secure Network Setup With AWS

As developers, we often want to gain practical experience with core AWS services and concepts by building a POC or running experiments. This helps reinforce our learning and prepare us for real-world challenges. Today, I'm sharing my learning and experience on AWS networking, especially around setting up a custom AWS VPC that prioritises secure access and network isolation.

Given that I need to deploy a small Proof of Concept (POC) of my **expense tracking app,** which helps me track my monthly spending and provides suggestions for saving, turning financial insights into actionable habits, there are challenges in terms of balancing **speed of development** with **security best practices**. 

While it's tempting to put everything in a single public subnet or using AWS default resources (such as default VPC, NACL, Security Group, etc.) to get the app running quickly, doing so exposes sensitive data—like my expense records—and increases the risk of misconfigurations.

My goal is to design a **lean and secure AWS network** that allows me to:

1. Deploy the app in a controlled environment.
2. Restrict access to sensitive components such as the database.
3. Enable safe developer access for ongoing development and experimentation.

I'll provide a clear, step-by-step guide for building secure, practical networking on AWS—even for small personal projects and eventually a high available and resilient architecture for enterprise applications.

![Simple and Secure Network Setup](/images/blogs-images/simple-secure-vpc-network-setup.png)

---

## **1. Split Your Network into 3 Layers**

Even for a small POC, you should separate what faces the internet from what stores data.

Think of your VPC as three zones:

#### **Public Subnet – The Front Door**

This is where things that must talk to the internet live:

- Application Load Balancer (ALB)
- Bastion (jump box for SSH access)

---

#### **Private Subnet – Your App**

This is where your app runs. AWS offers a wide spectrum of computing services, ranging from VMs to serverless and container services. Depending on your app's needs, you might choose one based on how much control you want over the underlying infrastructure:

- **Virtual Machines**: EC2, Lightsail
- **Serverless Compute**: Lambda, Fargate, App Runner
- **Container Orchestration**: ECS, EKS

These **do not** have public IPs.
They only accept traffic from the ALB or the Bastion.
For the purpose of my POC, the expense app is running on EC2 instances

---

#### **Database Subnet – Your Data**

This is the most protected layer. The database subnet has **no route to the internet** at all. AWS offers different engines optimised for specific data shapes (Relational, Key-value, Document, etc.):

- **Relational**: RDS (MySQL, PostgrSQL, MariaDB, Oracle, SQL Server), Aurora
- **Key-value & Document (NoSQL)**: DynamoDB, DocumentDB
- **In-Memory DBs**: Elasticache, MemoryDB for Redis
- **Specialised & Purpose-Built**: Redshift, Neptune, Timestream, LedgerDB

**Why this matters:**

Even if someone hacks your app, they still can't directly reach your database.

---

## **2. Control Traffic with NACL & Security Groups**

#### Network Access Control List (NACL)

The **NACLs** act as a second, broader layer of defence that sits at the **subnet boundary**.

A NACL acts like a security checkpoint at the entrance to the entire neighbourhood (the subnet).

In my 3-segment setup (Public, Private, Database), the NACL serves three critical roles:

- **Stateless Filtering:** Unlike Security Groups, NACLs are **stateless**. This means if you allow traffic *in* (Inbound), you must explicitly allow the response traffic *out* (Outbound). This is a rigid, "zero-trust" way to ensure no data leaves a subnet unless specifically permitted.
- **The "Deny" List:** NACLs allow you to write **"Deny" rules**. If you see malicious traffic coming from a specific IP address, you can block it at the NACL level before it even touches your load balancer or instances.
- **Subnet-Level Isolation:** You can use NACLs to ensure that the **Database Subnet** cannot, under any circumstances, communicate with the **Internet Gateway**, even if someone accidentally misconfigures a Security Group.

#### Security Group

Security Groups act like firewalls for each layer.
Here's the simple rule set:

**ALB Security Group**

- Allow inbound: `80` / `443` from the internet

**EC2 Security Group**

- Allow inbound: only from ALB
- Allow SSH: only from Bastion

**Database Security Group**

- Allow inbound: only from EC2

Nothing else can talk to your database.

No IP ranges. No internet. Only your app.

### NACL vs. Security Group

The **NACL** is the security gate at the entrance to the entire neighbourhood (the subnets), while the **Security Group** is the smart lock on the front door of your specific house (the EC2 instance).

**Security Group is Stateful:** If you send a request out from your app to an API, the Security Group "remembers" that request and automatically allows the response back in, regardless of inbound rules.

**NACL is Stateless:** It has no memory. If you allow traffic *in* on port 80, you must also create an explicit rule to allow that traffic *out* on the ephemeral ports; otherwise, the connection will be dropped.

**Security Group:** Evaluates **all rules** before deciding whether to allow traffic. There is no concept of "order." If any rule matches and allows the traffic, it gets through.

**NACL:** Evaluates rules in **numerical order**, starting from the lowest number. As soon as a rule matches (whether it's an "Allow" or a "Deny"), it stops looking at the rest of the list.

Using the NACL & Security Group provides **Defence in Depth**. If a developer accidentally opens an EC2 Security Group to the whole world, the NACL acts as the safety net that still blocks unauthorised traffic at the subnet perimeter.

---

## **3. Use a Bastion for Admin Access**

A **Bastion Host** (also called a "Jump Box") is a hardened server in a **Public Subnet** that serves as a secure gateway for developers to access instances in **Private Subnets**.

Think of it as a controlled entry point. Since your application servers live in a private subnet with no direct internet access, you need a way to manage them safely.

**How the "Jump" Works**

To log in to your app server, you use a two-hop SSH connection:

1. **Step 1:** You SSH from your laptop into the **Bastion Host** (using its Public IP).
2. **Step 2:** Once "inside" the Bastion, you initiate a second SSH connection to the **App Server** (using its Private IP).

Lock it down:

- Only allow **your home or corporate IPs** on port 22
- Never use `0.0.0.0/0`

This gives you a single, secure entry point.

---

## **4. Let Your App Reach AWS Services Privately**

**AWS PrivateLink** is a technology that allows you to connect your VPC to AWS services (like S3 or DynamoDB) or third-party services as if they were sitting right inside your own private network.

Normally, AWS services like S3 and DynamoDB live on the "public" side of AWS. To reach them from a private subnet, your traffic would usually have to:

1. Exit your private subnet.
2. Pass through a **NAT Gateway**.
3. Travel across the **Public Internet** (even if it's within the AWS global network).
4. Reach the service endpoint.

**PrivateLink changes this.** It places a "virtual network card" (Elastic Network Interface) directly into your private subnet. Your app then talks to this internal IP address, and the traffic never leaves the AWS private network.

Here is why it is recommended for connecting to other AWS services:

- **Zero Internet Exposure:** Because you are using PrivateLink, your private EC2 instances do not need a NAT Gateway or an Internet Gateway to store files in S3 or query DynamoDB. This makes your data much harder to be intercepted.
- **Simplified Firewall Rules:** You don't have to manage complex "allow" rules for massive ranges of public AWS IP addresses. You only need to allow traffic to the specific internal IP of the VPC Endpoint.
- **Reduced Data Costs:** While there is a small hourly fee for the endpoint, you often save money on "Data Transfer Out" charges that occur when traffic goes over the public internet.

For more use cases, check out AWS Private Link document: https://docs.aws.amazon.com/vpc/latest/privatelink/what-is-privatelink.html

---

## **5. Handling Outbound Internet**

In a standard AWS setup, an instance in a private subnet is "dark." It has no route to the internet. A **NAT Gateway** usually acts as the bridge that allows those private instances to "talk out" (for updates or API calls) without letting the internet "talk in."

Even with VPC Endpoints (mentioned above), your app might still be "stranded" in the private subnet if it needs to do any of the following:

- **Software Updates:** Running `bundle install`, `yum update`, or `apt-get upgrade` requires reaching external repositories (RubyGems, GitHub, Ubuntu mirrors).
- **Third-Party APIs:** If your app connects to **Stripe** for transaction data or **SendGrid** for emails, those are public internet endpoints. A VPC Endpoint cannot reach them.
- **External OAuth:** If you want users to "Login with Google," your server needs to talk to Google's servers.

However, NAT Gateway will cost ~32$/month, for a cheaper option, you can use a **NAT Instance** (a tiny `t4g.nano` instance you manage yourself in case of setting this up for a POC or experiments). It's significantly cheaper than the managed NAT Gateway service, though it requires more manual setup.

---

## **6. Use IAM Roles, Not Access Keys**

Never put AWS keys inside your app!

An **IAM Role** is the "identity card" for your application. It allows your app to prove to AWS that it has permission to read or write data without you ever having to handle secret passwords or access keys.

If you didn't use an IAM Role, you would have to manually paste your **AWS Access Key** and **Secret Key** into your app credentials file. If someone steals your code or accesses your server, they get your keys. Those keys are permanent until you manually change them.

With an IAM Role, your EC2 instance uses **temporary credentials** that AWS rotates automatically every few hours. Even if a hacker compromised the temporary token, it would expire shortly after.

You can create an **IAM Role** with a specific policy (e.g., `Allow S3:PutObject to My-Expense-Bucket`). You then attach this role to your EC2 instance. This attachment is called an **Instance Profile**.

When your app code uses the AWS SDK to upload a data to S3, the SDK automatically asks the **EC2 Metadata Service** for credentials. Because the Instance Profile is attached, the Metadata Service hands back a temporary token that the SDK uses to sign the request.

Using IAM Roles give you:

- **Easier Deployment:** You don't have to manage environment variables for your keys across different servers.
- **Least Privilege:** You can restrict your app to *only* touch the specific S3 bucket for your app, rather than your entire AWS account.
- **Auditability:** Every action taken by the EC2 instance is logged in **AWS CloudTrail** under the name of that specific IAM Role, so you know exactly what your app is doing.

---

## **Final Result**

With this setup, your POC has:

- A **public front door** (ALB)
- A **private application layer**
- A **fully isolated database**
- **Controlled SSH access**
- **Private access to AWS services**

You're still moving fast — but now you're building on a foundation that won't fall apart if your app becomes the real thing.

## Step-by-Step: Creating the 3-Tier Secure VPC

1. **Open the VPC Wizard:** Go to the [Amazon VPC console](https://console.aws.amazon.com/vpc/) , under **Your VPCs**, choose **Create VPC**.
2. **Select Strategy:** Under **Resources to create**, choose **VPC and more** (instead of **VPC only**).
    - *Use Case:* This creates the VPC, Subnets, Gateways, and Route Tables in a single workflow.
3. **Name and CIDR:** Provide a Name tag (e.g., `ExpenseTracker-VPC`) and set your **IPv4 CIDR block** to `10.0.0.0/16`.
    - *Best Practice:* Ensure your CIDR block is large enough to allow for future subnet expansion. See https://cidr.xyz/ for CIDR calculation.
4. **Availability Zones (AZs):** For a simple POC, choose **1 AZ** (matching `us-east-1a` ).
    - *Best Practice:* For production, always choose **2 or more AZs** to ensure high availability.
5. **Configure the 3-Tier Subnets:** * **Number of Public subnets:** Set to **1**. (e.g. CIDR: `10.0.0.0/24`)
    - **Number of Private subnets:** Set to **2**.
        - Subnet 1 will be your **Application Tier** (e.g. CIDR: `10.0.1.0/24`).
        - Subnet 2 will be your **Database Tier** (e.g. CIDR: `10.0.1.1/24`).
    - *Best Practice:* Physically separating the Application and Database tiers into different subnets allows you to apply strict NACL boundaries between them.
6. **NAT Gateways:** Select **Regional - new** which offers a multi-AZ NAT Gateway or **Zonal** which requires 1 NAT Gateway per AZ. In this example, we'll choose **Zonal**, and select **1 per AZ**.
    - *Use Case:* Required for your EC2s in the private subnet to download external libraries/packages or security patches from the internet.
7. **VPC Endpoints (The Private Link):**  Select **S3 Gateway**.
    - *Best Practice:* Gateway endpoints are free and keep S3 traffic off the internet.
    - *Manual Step:* You will need to manually add the **Interface VPC Endpoints** (for other services) as the wizard primarily defaults to S3 Gateway.
8. **DNS Options:** Ensure **Enable DNS hostnames** and **Enable DNS resolution** are both checked.
    - *Use Case:* Required for the Interface VPC Endpoints to resolve correctly within your private subnets.
9. **Review the Resource Map:** In the **Preview** pane, verify that:
    - The **Public Route Table** points to the **Internet Gateway**.
    - The **Private Route Table** points to the **NAT Gateway** and **S3 Gateway**.
10. **Finalise:** Choose **Create VPC**.

---

### Post VPC Creation

Once the wizard finishes, you must perform these two manual steps to complete the firewalls setup:

**1. Configure the NACL (The Purple Boundary)**

The wizard creates a default "Allow All" NACL for the above VPC. To restrict further access to each subnet, go back to the main **VPC Console**, then go to **Network ACLs** under **Security**:

- **Create network ACL:**
    - `private-nacl` to *only* allow inbound traffic from the **Public Subnet** (`10.0.0.0/24`) and deny all outbound internet traffic and associate the **Private Subnet** with it.
    - `database-nacl` to *only* allow inbound traffic from the **Private Subnet** (`10.0.1.0/24`) and deny all outbound internet traffic and associate the Database **Subnet** with it.

**2. Create Security Groups (The Red Boundaries)**

You must manually create the four distinct Security Groups shown in the red dashed boxes:

- **ALB SG:** Allows port 80/443 from `0.0.0.0/0`.
- **Bastion SG:** Allows port 22 from **only your home IP or corporate IPs**.
- **App SG:** Allows port 80 from the **ALB SG** and port 22 from the **Bastion SG**.
- **DB SG:** Allows the database port (e.g., 5432 for Postgres) **only** from the **App SG**.

By following this blueprint, you've successfully deployed a 'dark' VPC—a robust environment where your application logic and data remain invisible to the public web while your app's functionalities are still securely published to the world. You're operating on a foundation built for privacy and resilience.
And if you're ready to take it to the next level, check out my follow-up blog on scaling this architecture across multiple Availability Zones for enterprise-grade reliability.

Happy building! 🚀