# **AWS Project: Hosting a Standalone Web Application with Custom VPC Setup**

This project demonstrates how to deploy a standalone web application built with HTML and CSS on AWS. The application is hosted in a custom VPC (Virtual Private Cloud) with private and public subnets, and it is accessible through a custom domain. The project also covers configuring networking components such as route tables, internet gateways, and NAT gateways for secure and scalable hosting.

---

## **Overview**

### **Project Description**
- **Web Application**: A standalone application using HTML and CSS.
- **Custom Domain**: The application is accessible via a custom domain.
- **AWS Networking Setup**:
  - **Custom VPC**: Used to host and isolate resources.
  - **Subnets**: Public and private subnets for controlled access.
  - **Route Tables and Gateways**: Configured for traffic routing and internet access.
  - **EC2 Instance**: Deployed to serve the web application.

### **AWS Components Used**
- **Virtual Private Cloud (VPC)**: Isolated network for resources.
- **Subnets**: Logical IP address groupings for public and private resources.
- **Route Tables**: Manage traffic routing within the VPC.
- **Internet Gateway (IGW)**: Allows public access to resources.
- **NAT Gateway**: Enables private subnet resources to access the internet.
- **Elastic Compute Cloud (EC2)**: Virtual server hosting the application.

---

## **Steps to Configure and Deploy**

### **1. Create a Custom VPC**
A Virtual Private Cloud (VPC) is the foundational network for your AWS resources. It defines an isolated network environment.

- **VPC Name**: `MyVPC`
- **IP Address Range**: `20.0.0.0/16`
  - Total number of IPs: \( 2^{(32-n)} \) (where \( n \) is the CIDR range).
    - For `/16`, \( 2^{(32-16)} = 65,536 \) IPs.
  - AWS reserves **5 IPs per subnet** for internal use:
    - 1st IP: Network address.
    - 2nd IP: Reserved for AWS DNS.
    - 3rd IP: Reserved for future use.
    - Last IP: Broadcast address.
    - 1 additional reserved IP.
- **DNS Hostnames**: Enable DNS hostname resolution to identify instances by names.

---

### **2. Create Subnets**
Subnets are logical groupings of IP addresses within the VPC. They allow you to isolate resources based on accessibility requirements.

- **Subnet Types**:
  - **Public Subnet**: Accessible from the internet. Used for resources like web servers.
  - **Private Subnet**: Not directly accessible from the internet. Used for internal services like databases or backend systems.

- **High Availability**:
  - Create subnets across **multiple Availability Zones (AZs)** to ensure fault tolerance.
  - Example: One public subnet in `us-east-1a` and another in `us-east-1b`.

- **Subnet Calculations**:
  - Use online CIDR calculators to divide the VPC’s IP address range into subnets based on requirements.

- **Public vs. Private Decision**:
  - Public subnets allow auto-assignment of public IPs:
    - Configure in **Subnet Settings → Enable Auto-assign public IP**.

---

### **3. Create and Configure Route Tables**
Route tables define how traffic flows within the VPC and to external networks.

- **Steps**:
  1. Create two route tables:
     - One for the public subnet.
     - One for the private subnet.
  2. **Public Route Table**:
     - Add a route for internet traffic (`0.0.0.0/0`) via the **Internet Gateway (IGW)**.
  3. **Private Route Table**:
     - Add a route for internet-bound traffic via the **NAT Gateway**.

- **Subnet Associations**:
  - Associate the public route table with public subnets.
  - Associate the private route table with private subnets.

---

### **4. Create an Internet Gateway**
The Internet Gateway (IGW) allows resources in the public subnet to communicate with the internet.

- **Steps**:
  1. Create an Internet Gateway.
  2. Attach the IGW to the VPC.
  3. Update the public route table:
     - Add a route for all outbound traffic (`0.0.0.0/0`) via the IGW.

---

### **5. Deploy an EC2 Instance**
An EC2 instance is used to host and serve the web application.

- **Instance Configuration**:
  - **AMI**: Ubuntu.
  - **Instance Type**: `t2.micro` (free-tier eligible).
  - **Key Pair**: Create a key pair for secure SSH access.
  - **Network**: Choose the VPC and public subnet.
  - **Security Group**: Allow HTTP, HTTPS, and SSH traffic.
  - **Launch**: Deploy the instance and note its public IP address.

- **Connect to the Instance**:
  - Use SSH to connect:  
    ```bash
    ssh -i <key-pair>.pem ubuntu@<public-ip-address>
    ```

---

### **6. Enable Private Subnet Internet Access**
Resources in the private subnet cannot directly access the internet. To enable outbound communication (e.g., software updates), configure a **NAT Gateway**.

- **Steps**:
  1. Create a NAT Gateway in the **public subnet**.
  2. Assign an Elastic IP to the NAT Gateway.
  3. Update the private route table:
     - Add a route for internet-bound traffic (`0.0.0.0/0`) via the NAT Gateway.

---

### **7. Host the Web Application**
- Upload the HTML and CSS files to the EC2 instance:
  ```bash
  scp -i <key-pair>.pem index.html style.css ubuntu@<public-ip-address>:/var/www/html/

---

## **Deploying the Application**

This section outlines the process of deploying a static web application using a web server, configuring a custom domain, and making it accessible publicly.

---

### **1. Install a Web Server**
A web server hosts and serves static web applications, such as HTML and CSS files. Examples of web servers include **Apache2** and **NGINX**.

#### **Why Do You Need a Web Server?**
- Without a web server, accessing the public IP of your EC2 instance will not show any content.
- A web server processes HTTP requests and serves static files like HTML, CSS, and JavaScript to users.

#### **Steps to Install and Configure Apache2**
1. **Update the EC2 Instance**:
   - Always ensure your instance has the latest updates:
     ```bash
     sudo apt update && sudo apt upgrade -y
     ```

2. **Install Apache2**:
   - Apache2 is a widely used web server for hosting static content:
     ```bash
     sudo apt install apache2 -y
     ```

3. **Manage the Apache2 Service**:
   - Start the Apache2 service:
     ```bash
     sudo systemctl start apache2
     ```
   - Stop the Apache2 service:
     ```bash
     sudo systemctl stop apache2
     ```
   - Enable the Apache2 service to start automatically on boot:
     ```bash
     sudo systemctl enable apache2
     ```

---

### **2. Deploy Your Application Code**
Your application code is stored in a GitHub repository. Follow these steps to deploy it to the Apache2 server:

1. **Install Git**:
   - Git is required to clone the repository containing your application code:
     ```bash
     sudo apt install git -y
     ```

2. **Clone the GitHub Repository**:
   - Replace `<repository-url>` with the actual URL of your repository:
     ```bash
     git clone <repository-url>
     ```

3. **Move Application Files to the Apache2 Hosting Directory**:
   - By default, Apache2 serves files from `/var/www/html/`. Move your HTML and CSS files there:
     ```bash
     sudo mv ./* /var/www/html/
     ```

4. **Access the Application**:
   - Open a web browser and navigate to the public IP address of your EC2 instance. You should see your deployed application.

---

### **3. Use a Custom Domain Name**

#### **Why Use a Custom Domain Name?**
Accessing your application using a domain name (e.g., `example.com`) is more professional and user-friendly than using the public IP address.

#### **Steps to Configure a Custom Domain**
1. **Purchase a Domain Name**:
   - Purchase a domain from a registrar like **GoDaddy** or another provider.

2. **Set Up AWS Route 53**:
   - Route 53 is AWS's DNS service used to manage domain name mappings.

   **Steps**:
   - Navigate to the Route 53 service in the AWS Management Console.
   - **Create a Hosted Zone**:
     - **Domain Name**: Enter your purchased domain name (e.g., `example.com`).
     - **Type**: Select **Public Hosted Zone**.
   - Route 53 automatically creates two records:
     - **NS Record**: Specifies the name servers that route traffic for the domain.
     - **SOA Record**: Contains administrative information about the domain.

3. **Update Name Servers on the Domain Registrar**:
   - Go to your domain registrar (e.g., GoDaddy).
   - Replace the registrar's default name servers with the Route 53 **NS Record** values.

4. **Add DNS Records in Route 53**:
   - **A Record**:
     - Maps the domain name to the public IP address of your EC2 instance.
     - Example configuration:
       - **Record Type**: `A`
       - **Value**: EC2 instance public IP address.
       - **TTL**: 300 seconds (default).
       - **Routing Policy**: Simple routing.
   - **CNAME Record** (Optional):
     - Use this to map subdomains (e.g., `www.example.com`) to your main domain.

5. **Test the Domain**:
   - Copy your domain name (e.g., `example.com`) and search it in a browser.
   - Your web application should load successfully.

---

### **4. Verify the Domain Configuration**
You can use tools like **whois domain info** to verify your domain setup:
- Check the **IP address**, **name servers**, and **hosting provider**.
- Ensure the domain points correctly to the EC2 instance.

---

### **Key Notes**
1. **Apache2 Configuration**:
   - Place the `index.html` file in `/var/www/html/` for Apache2 to serve the application.
2. **DNS Configuration**:
   - Ensure the Route 53 hosted zone and your domain registrar are properly configured.
3. **Testing**:
   - Test the application using both the EC2 public IP and the custom domain.

---

### **Conclusion**
This setup provides a complete solution for hosting and deploying a static web application on AWS. By using a web server and a custom domain, your application becomes accessible and professional for end-users.

Include screenshots to demonstrate:
- Apache2 installation and configuration.
- Route 53 hosted zone and record setup.
- Testing the application via both IP and domain name.
