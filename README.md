# Azure Virtual Network & NSG Rules for Mini IT Infrastructure  

## **Project Overview**  
This project involves setting up a **secure and scalable Azure Virtual Network (VNet)** for a **mini IT infrastructure** consisting of:  

- **Web Server** (Frontend) 🌐  
- **Application Server** (Middleware) 🖥️  
- **Database Server** (Backend) 🗄️  
- **Jump Server** (For Secure Remote Access) 🔐  

To ensure **network segmentation, security, and controlled access**, **Network Security Groups (NSGs)** and **Application Security Groups (ASGs)** were implemented to enforce **firewall rules, restrict unauthorized traffic, and allow only necessary communications** between the servers.  

---

## **📌 Architecture Design**  

### **1️⃣ Azure Virtual Network (VNet) Configuration**  
✅ **VNet Name:** `PJ-SEC-Vnet`  
✅ **Address Space:** `10.0.0.0/16`  
✅ **Subnets:**  
   - `Web-Subnet (10.0.1.0/24)` – Public-facing web server  
   - `App-Subnet (10.0.2.0/24)` – Internal application server  
   - `DB-Subnet (10.0.3.0/24)` – Secured database layer  
   - `Jump-Subnet (10.0.4.0/24)` – Jump server for secure remote management  


<h3>📸 Screenshot: Virtual Network & Subnet Configuration</h3>
<img src="assets/Screenshot 2025-03-24 at 08.09.59.png" alt="Virtual Network & Subnet Configuration" width="600">

---

## **💻 Virtual Machines (VMs) Deployment**  

The following VMs were deployed to support the mini IT infrastructure:  

| **VM Name**        | **Purpose**            | **OS**         | **Subnet**         | **NSG Applied**      |
|------------------|--------------------|--------------|----------------|-----------------|
| **web-server1**   | Web Server (Frontend)   | Ubuntu 22.04 | Web-Subnet (10.0.1.0/24)  | PJ-SEC-NSG (Web-ASG) |
| **applicationserver** | Application Server     | Ubuntu 22.04 | App-Subnet (10.0.2.0/24)  | PJ-SEC-NSG (App-ASG) |
| **db-server1**    | Database Server        | Ubuntu 22.04 | DB-Subnet (10.0.3.0/24)   | PJ-SEC-NSG (DB-ASG) |
| **jump-srv**      | Secure Admin Access    | Ubuntu 22.04 | Jump-Subnet (10.0.4.0/24) | PJ-SEC-NSG (Admin) |

📌 **Security Configurations Applied:**  
✅ **web-server1:** Runs **Nginx/Apache**, only accessible on ports **80 & 443** from the internet.  
✅ **applicationserver:** Processes backend logic, only accessible from **web-server1** on ports **5000, 8080**.  
✅ **db-server1:** Stores critical data, **no public access**, only reachable from **applicationserver**.  
✅ **jump-srv:** **The only VM with SSH open** (Port 22), accessible only from an admin's public IP for secure access.  

  
<h3>📸 Screenshot: Virtual Machines Deployed in Azure</h3>
<img src="assets/Screenshot 2025-03-24 at 08.48.02.png" alt="Virtual Machines Deployed in Azure" width="600">


---

## **2️⃣ Network Security Group (NSG) & Application Security Group (ASG) Rules**  

#### **🔹 Application Security Groups (ASGs) Used**  
To simplify NSG rules and reduce manual IP-based configurations, I implemented **Application Security Groups (ASGs):**  

- **`web-ASG`** → Web servers  
- **`app-ASG`** → Application servers  
- **`db-ASG`** → Database servers  

#### **🔹 Network Security Group (NSG) Rules - `PJ-SEC-NSG`**  

✅ **web-ASG (Web-Subnet)**  
- Allow HTTP (Port 80) & HTTPS (Port 443) from the internet  
- Allow SSH (Port 22) only from the Jump Server  
- Block all other inbound traffic  

✅ **app-ASG (App-Subnet)**  
- Allow inbound traffic from web-ASG (Port 5000, 8080)  
- Allow SSH only from Jump Server  
- Deny all internet access except for patching  

✅ **db-ASG (DB-Subnet)**  
- Allow MySQL (Port 3306) / PostgreSQL (Port 5432) from app-ASG only  
- Block all direct internet access  
- Allow SSH from Jump Server for maintenance  

✅ **jump-srv (Jump-Subnet)**  
- Allow SSH (Port 22) only from Admin Public IP  
- Allow outbound SSH to Web, App, and DB subnets  

 
<h3>📸 Screenshot: Network Security Group (NSG) Rules</h3>
<img src="assets/Screenshot 2025-03-24 at 08.11.01.png" alt="Network Security Group (NSG) Rules" width="600">


🚀 **Result:** **Zero Trust Network Model** applied—only required communications allowed!  

---

## **🔍 Key Findings & Security Enhancements**  

### **1️⃣ Segmented Network for Security**  
🚀 **Isolated layers** between Web, App, and DB ensure attackers cannot jump across networks.  
🔐 **NSGs enforce least privilege**—only necessary traffic is allowed.  

### **2️⃣ Application Security Groups (ASGs) Simplify Access Control**  
✅ **Instead of managing individual IP addresses, ASGs allow flexible role-based access**.  
✅ **When new servers are added to ASGs, they automatically inherit security rules**.  

### **3️⃣ Secure Remote Access via Jump Server**  
✅ **No direct SSH access** to any production server from the internet.  
✅ **All administrative access is funneled through the Jump Server**, reducing attack surface.  

 
<h3>📸 Screenshot: Secure Jump Server Access</h3>
<img src="assets/Screenshot 2025-03-24 at 08.52.09.png" alt="Jump Server Access" width="600">


---

## **📊 Kusto Query (KQL) for Monitoring Logs in Log Analytics**  
To detect **suspicious traffic or NSG rule violations**, we can use **Azure Log Analytics**:  

```kql
AzureNetworkAnalytics_CL 
| where Subnet == "Web-Subnet"
| where Action_s == "Deny"
| summarize BlockedAttempts=count() by SourceIP
| order by BlockedAttempts desc
