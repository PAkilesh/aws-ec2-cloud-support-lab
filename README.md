# AWS EC2 Cloud Support & Monitoring Lab

## Project Overview

Deployed and managed a Linux-based web server on Amazon Web Services (AWS), configured secure remote access, hosted a custom website using Nginx, performed infrastructure health checks, analyzed web server logs, and configured Amazon CloudWatch CPU monitoring.

This project demonstrates hands-on skills relevant to Cloud Support Engineer, Linux Administrator, and L1 Infrastructure Support roles.

## Architecture

Internet User → AWS Security Group → EC2 Instance (Amazon Linux 2023) → Nginx Web Server → HTML Website

Monitoring: EC2 Metrics → Amazon CloudWatch → CPU Utilization Alarm → Amazon SNS Email Notifications

## AWS Services and Technologies

- Amazon EC2 — Virtual server deployment
- Amazon VPC — Networking and subnet configuration
- EC2 Security Groups — Inbound and outbound traffic control
- Amazon EBS — Persistent storage
- Amazon CloudWatch — Infrastructure monitoring and alarms
- Amazon SNS — Email notification configuration
- Amazon Linux 2023 — Linux operating system
- Nginx — HTTP web server
- SSH — Secure remote administration
- Linux CLI — System administration and troubleshooting

## Implementation

### 1. EC2 Deployment

- Launched a `t3.micro` instance in the Mumbai region (`ap-south-1`).
- Selected Amazon Linux 2023.
- Configured an 8 GiB encrypted gp3 EBS root volume.
- Used the default VPC and subnet.
- Created an SSH key pair for secure authentication.

### 2. Network Security

Configured the EC2 Security Group:

| Protocol | Port | Source | Purpose |
|---|---|---|---|
| SSH | 22 | Administrator public IP `/32` | Restricted remote administration |
| HTTP | 80 | `0.0.0.0/0` | Public website access |

### 3. SSH Connectivity

Connected from Windows PowerShell:

```bash
ssh -i "path/to/private-key.pem" ec2-user@EC2_PUBLIC_IP
```

Verified the server using:

```bash
hostname
whoami
cat /etc/os-release
ip -4 addr show
df -h
```

### 4. Nginx Web Server

Installed and configured Nginx:

```bash
sudo dnf install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl status nginx --no-pager
```

Deployed a custom HTML webpage under:

```text
/usr/share/nginx/html/index.html
```

Verified the web server:

```bash
curl -I http://localhost
```

Result: `HTTP/1.1 200 OK`.

### 5. Linux Health Checks

Executed:

```bash
uptime
free -h
df -h /
systemctl is-active nginx
curl -o /dev/null -s -w "HTTP Status: %{http_code}\n" http://localhost/
```

Observed during the lab:

- System load average: `0.00, 0.00, 0.00`
- Available RAM: approximately 592 MiB
- Root disk usage: 21%
- Nginx service: active
- HTTP response: 200 OK

### 6. Nginx Log Analysis

Reviewed access and error logs:

```bash
sudo tail -n 20 /var/log/nginx/access.log
sudo tail -n 20 /var/log/nginx/error.log
sudo tail -f /var/log/nginx/access.log
```

Identified successful HTTP 200 requests and HTTP 404 responses for missing `favicon.ico` and `robots.txt` files.

Observed requests identifying themselves as automated web crawlers.

### 7. Amazon CloudWatch Monitoring

Reviewed EC2 metrics including:

- CPUUtilization
- NetworkIn
- NetworkOut
- NetworkPacketsIn
- CPUCreditBalance

Created a CloudWatch alarm:

| Configuration | Value |
|---|---|
| Alarm name | `EC2-High-CPU-Alert` |
| Metric | CPUUtilization |
| Statistic | Average |
| Period | 5 minutes |
| Threshold | Greater than 80% |
| Evaluation | 3 out of 3 datapoints |
| Missing data | Treat missing data as missing |

The alarm transitioned from `INSUFFICIENT_DATA` to `OK` after evaluation.

Configured an Amazon SNS topic for email notifications. Email delivery depends on confirmation of the SNS subscription.

## Troubleshooting Case Studies

### Case 1 — SSH Connection Timeout

**Issue:** SSH connection timed out when connecting to EC2.

**Investigation:**
- Checked the EC2 Security Group SSH rule.
- Verified the administrator's public IP.
- Inspected subnet routing and internet gateway configuration.
- Reviewed network ACL rules.
- Checked EC2 instance status and system logs.

**Root cause:** An incorrect destination public IP address was used. The SSH command omitted the final digit of the EC2 public IP.

**Resolution:** Corrected the destination IP and established SSH connectivity successfully.

**Learning:** Always verify the exact destination address before troubleshooting more complex network components.

### Case 2 — Browser Displayed an Old Webpage

**Issue:** Browser continued showing the default Nginx welcome page after deploying custom HTML.

**Investigation:**
- Verified the updated `index.html` file.
- Confirmed Nginx served the updated HTML using `curl`.
- Compared HTTP response content with the browser output.

**Resolution:** Opened the website using a cache-busting query parameter (`?v=2`), and the updated page appeared.

**Learning:** Distinguish between server-side content issues and client-side caching.

### Case 3 — HTTP 404 Entries in Nginx Logs

**Issue:** Nginx error logs contained missing-file errors.

**Investigation:** Reviewed HTTP requests for `favicon.ico` and `robots.txt`.

**Finding:** These files were not present in the website directory.

**Learning:** Not every HTTP 404 indicates a web application outage. Analyze the requested resource and impact before escalating.

## Skills Demonstrated

AWS EC2, VPC Networking, Security Groups, Linux Administration, SSH, Nginx, HTTP Troubleshooting, Log Analysis, CloudWatch Monitoring, SNS, Server Health Checks, and Incident Troubleshooting.

## Security Notes

- SSH access restricted to the administrator's public IP.
- Private SSH keys and AWS credentials are not stored in this repository.
- Public HTTP hosting is for demonstration only; HTTPS should be configured for production use.
- AWS resources should be stopped or removed when no longer required to minimize costs.
