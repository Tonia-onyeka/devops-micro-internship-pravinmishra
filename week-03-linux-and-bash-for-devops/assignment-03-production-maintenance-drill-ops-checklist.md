# Assignment 3 — Production Maintenance Drill (OPS Checklist)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will treat your already deployed React application (on Ubuntu VM with Nginx) as a live production system. You will perform structured operational checks covering network validation, service health, log analysis, resource monitoring, configuration verification, and incident simulation with recovery — mirroring real on-call DevOps responsibilities.

---

# Task 1 — Server Access & Networking Validation

## Goal
 
Verify that the deployed React application is reachable from the browser and confirm basic network connectivity of the Ubuntu VM.

### Evidence

#### Screenshot 1 — Browser showing the React app with your Full Name visible on the UI

Add your screenshot here.

---

#### Screenshot 2 — Output of `ip a`

Add your screenshot here.

---

#### Screenshot 3 — Output of `sudo ss -tulpen`

Add your screenshot here.

---

#### Screenshot 4 — Output of `sudo ufw status`

Add your screenshot here.

---

### Notes

Answer the following in your own words:

**1. What proves Nginx is listening on 0.0.0.0:80?**

The sudo ss -tulpen command shows that Nginx is listening on port 80 with the address 0.0.0.0:80. The 0.0.0.0 address means the web server is accepting HTTP connections from all network interfaces, making it accessible to users over the internet. Since the process name is nginx, it confirms that Nginx is the service using port 80.

---

**2. What proves SSH is active on port 22?**

The command output also displays 0.0.0.0:22 with the process name sshd. This indicates that the SSH service is running and listening on port 22 across all network interfaces. As a result, administrators can securely connect to the EC2 instance remotely using SSH.

---

**3. Did you find any unexpected open ports? Explain briefly.**

No unexpected ports were found. The only services exposed to external users are Nginx on port 80 and SSH on port 22, which are required for hosting the web server and allowing remote access. Other services, such as Chrony and systemd-resolved, are bound to the local loopback address (127.0.0.1), meaning they can only be accessed from within the server and are not exposed to the internet. This confirms that the server is securely configured with only the necessary ports open.
---

# Task 2 — Service Health & Systemd Validation (Nginx)

## Goal

Verify that Nginx is properly installed, running, enabled at boot, and safely configured.

### Evidence

#### Screenshot 1 — Output of `systemctl status nginx --no-pager`

Add your screenshot here.

---

#### Screenshot 2 — Output of `sudo nginx -t`

Add your screenshot here.

---

#### Screenshot 3 — Output of `sudo ss -lptn '( sport = :80 )'`

Add your screenshot here.

---

### Notes

Answer the following in your own words:

**1. What happens if Nginx fails to restart in production?**

If Nginx does not restart successfully, the website will become unavailable because Nginx is responsible for serving web traffic on port 80. Visitors trying to access the site will likely receive connection errors or timeouts since no service will be listening on that port. This is especially critical during deployments or configuration changes because it can result in website downtime until the issue is identified and resolved.

---

**2. What's your basic rollback plan?**

Before making any changes to the Nginx configuration, run sudo nginx -t to verify that the configuration file is valid and free of syntax errors. If the restart still fails, check the service status using systemctl status nginx --no-pager and review the detailed error logs with journalctl -u nginx --no-pager -n 50 to determine the cause of the failure. If the issue is due to an incorrect configuration, restore the last known working configuration from a backup or version control, test it again with sudo nginx -t, and then restart the service using sudo systemctl restart nginx. Maintaining a backup of the working configuration allows for a quick rollback and minimizes service downtime.

---

# Task 3 — Logs & Request Trace

## Goal

Verify real traffic flow and analyze logs to understand system behavior and errors.

### Evidence

#### Screenshot 1 — Output of `sudo tail -n 30 /var/log/nginx/access.log`

Add your screenshot here.

---

#### Screenshot 2 — Output of `sudo tail -n 30 /var/log/nginx/error.log`

Add your screenshot here.

---

#### Screenshot 3 — Output of `sudo journalctl -u nginx --no-pager -n 50`

Add your screenshot here.

---

### Notes

Answer the following in your own words:

**1. Were there any errors in the logs?**

- If yes, mention 1–2 example error lines from the logs and explain what each one means in simple terms.
- If no, explain what it means if the error log is empty or shows no recent errors during your check.

No, there were no errors in the Nginx error log during the check. Running the command sudo tail -n 30 /var/log/nginx/error.log returned no log entries, which means Nginx did not record any recent errors or failures. This indicates that the web server was operating normally at the time the logs were checked.


---

**2. If there were no errors, what does that indicate about the system?**

The absence of errors indicates that the Nginx service is running correctly and is not experiencing any recent issues. It suggests that the server is handling requests successfully, the configuration is working as expected, and there are no critical problems affecting the web service.

---

**3. Based on the access logs, were your curl requests visible in the log entries? What does that prove about traffic flow?**

Yes. The access logs include entries such as:

"GET / HTTP/1.1" 200 644 "-" "curl/8.18.0"

and

"HEAD / HTTP/1.1" 200 0 "-" "curl/8.18.0"

These entries confirm that the curl requests successfully reached the Nginx server and were processed correctly. The HTTP status code 200 indicates that the server responded successfully, proving that traffic is flowing properly from the client to the web server and that Nginx is serving requests as expected.

---

# Task 4 — System Resource Health Check (Capacity Red Flags)

## Goal

Assess server capacity and detect potential performance or failure risks.

### Evidence

#### Screenshot 1 — Output of `uptime`

Add your screenshot here.

---

#### Screenshot 2 — Output of `free -h`

Add your screenshot here.

---

#### Screenshot 3 — Output of `df -h`

Add your screenshot here.

---

#### Screenshot 4 — Output of `sudo du -sh /var/* | sort -h`

Add your screenshot here.

---

### Notes

Answer the following in your own words:

**1. Which resource looks most critical right now? (CPU/load, memory, or disk) Explain why.**

Based on the screenshots, disk usage appears to be the most critical resource to monitor. The df -h output shows that the root (/) filesystem is 60% full, leaving 2.7 GB of available space. Although the server still has sufficient free storage, disk space can fill up over time due to log files, application data, or system updates. The sudo du -sh /var/* | sort -h output also shows that the /var/lib (431 MB) and /var/cache (150 MB) directories are the largest consumers of storage in the /var directory. In comparison, the CPU load averages are 0.00, indicating no CPU stress, and the system has approximately 550 MB of available memory, so both CPU and memory are currently in a healthy state.

---

**2. What happens if disk becomes 100% full in a production server?**

If a production server's disk reaches 100% capacity, it can no longer write new data to the filesystem. This may prevent applications from creating or updating files, stop logs from being written, cause databases to fail, and prevent services such as Nginx from operating correctly. System updates and temporary file creation may also fail, leading to application crashes, service outages, or an unstable server. Monitoring disk usage and cleaning up unnecessary files before the disk becomes full helps maintain system reliability and prevents downtime.

---

# Task 5 — Configuration & Deployment Verification

## Goal

Ensure the correct React build is deployed and Nginx is serving it properly.

### Evidence

#### Screenshot 1 — Output of `ls -lah /var/www/html | head -n 20`

Add your screenshot here.

---

#### Screenshot 2 — Output of `grep -R "Deployed by" -n /var/www/html 2>/dev/null | head`

Add your screenshot here.

---

#### Screenshot 3 — Output of `grep -n "try_files" /etc/nginx/sites-available/default`

Add your screenshot here.

---

### Notes

Answer the following in your own words:

**1. How do you confirm that the correct version of the application is deployed?**

You can confirm the correct version is deployed by accessing the application and checking that it displays the expected version or recent changes. You can also verify the deployment by checking the deployment logs, the running container or service version, and ensuring the latest code or release is running successfully.

---

# Task 6 — Nginx Configuration Failure Simulation

## Goal

Simulate a real-world Nginx misconfiguration and recover the service safely.

### Evidence

#### Screenshot 1 — Output of `sudo nginx -t` showing the syntax error (broken config)

Add your screenshot here.

---

#### Screenshot 2 — Output of `sudo nginx -t` showing syntax ok (fixed config)

Add your screenshot here.

---

#### Screenshot 3 — Output of `curl -I http://<public-ip>` confirming recovery (200 OK)

Add your screenshot here.

---

### Notes

Answer the following in your own words:

**1. What caused the configuration failure?**

The configuration failure was caused by a syntax error in the Nginx configuration file. A required semicolon (;) was removed, making the configuration invalid and preventing Nginx from passing the configuration test.

---

**2. How did you fix the issue?**

I fixed the issue by correcting the syntax error, adding the missing semicolon back to the configuration file, and then running sudo nginx -t to confirm the configuration was valid before restarting the Nginx service.

---

**3. How can you avoid this kind of issue in real production systems?**

To avoid this issue, always test the Nginx configuration with sudo nginx -t before restarting the service. It's also a good practice to keep configuration files under version control, back up working configurations, and review changes before deploying them to production.

---

# Task 7 — Web Application Failure Simulation

## Goal

Simulate missing deployment content and recover the application safely.

### Evidence

#### Screenshot 1 — Output of `curl -I http://<public-ip>` showing failure (non-200 response)

Add your screenshot here.

---

#### Screenshot 2 — Output of `curl -I http://<public-ip>` confirming recovery (200 OK)

Add your screenshot here.

---

### Notes

Answer the following in your own words:

**1. What caused the application to break in this scenario?**

The application broke because the web content (the index.html file) was temporarily removed or renamed. As a result, Nginx could not find the webpage to serve, causing it to return a non-200 HTTP response.

---

**2. How did you fix the issue and restore the application?**

I restored the application by moving the index.html file back to its original location. After restoring the file, I verified the application with curl -I http://<public-ip>, which returned HTTP/1.1 200 OK, confirming the website was working again.

---

**3. What steps would you take to prevent this kind of issue in real production systems?**

To prevent this issue, I would verify deployment files before making changes, use version control and backups, test deployments in a staging environment, and validate the application after deployment using tools like curl or health checks. This helps detect and fix issues before they affect users.

---

# Task 8 — Security & Reliability Review

## Goal

Review and reflect on the security and reliability practices applied during this assignment.

### Security & Reliability Notes

Answer the following in your own words:

**1. Why is SSH key-based authentication more secure than sharing passwords?**

SSH key-based authentication is more secure because it uses a unique cryptographic key instead of a password. It is much harder to guess or crack and reduces the risk of unauthorized access.

---

**2. Why should only required ports be open on a production server?**

Only the required ports should be open to reduce the server's attack surface. Closing unnecessary ports helps prevent unauthorized access and improves the overall security of the server.

---

**3. Why is it important for Nginx to be enabled on boot?**

Enabling Nginx on boot ensures that the web server starts automatically whenever the server restarts. This keeps the website available without requiring manual intervention.

---

**4. What are the risks of sharing secrets, keys, or credentials publicly?**

Sharing secrets, SSH keys, or credentials publicly can allow attackers to gain unauthorized access to servers, applications, or cloud resources. This can lead to data breaches, service disruptions, or misuse of cloud infrastructure.

---

**5. Why should cloud resources be stopped or terminated when they are no longer needed?**

Stopping or terminating unused cloud resources helps reduce unnecessary costs, improves security by removing unused systems, and prevents resources from being accidentally exposed or misused.

---

# LinkedIn Post (Required)

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

`__________________________`

---

#### Screenshot — Published LinkedIn post

Add your screenshot here.

---

# Submission Instructions

- Add all required screenshots in your submission
- Full name must be visible in required screenshots
- Do not expose sensitive information (keys, passwords, account IDs)

---

# Completion Checklist

- [ ] Task 1: Screenshots (browser, ip a, ss -tulpen, ufw status) + Notes answered
- [ ] Task 2: Screenshots (nginx status, nginx -t, ss port 80) + Notes answered
- [ ] Task 3: Screenshots (access log, error log, journalctl) + Notes answered
- [ ] Task 4: Screenshots (uptime, free -h, df -h, du -sh) + Notes answered
- [ ] Task 5: Screenshots (ls html, grep deployed by, grep try_files) + Notes answered
- [ ] Task 6: Screenshots (nginx -t fail, nginx -t pass, curl recovery) + Notes answered
- [ ] Task 7: Screenshots (curl failure, curl recovery) + Notes answered
- [ ] Task 8: Security & Reliability Notes answered
- [ ] LinkedIn post published and URL submitted
- [ ] Full Name visible in all required screenshots
- [ ] No sensitive data exposed

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://pravinmishra.com/dmi  
- 🎓 DevOps for Beginners (Udemy): https://www.udemy.com/course/devops-for-beginners-docker-k8s-cloud-cicd-4-projects/  
- 🎓 Agentic AI DevOps with Claude Code: https://www.udemy.com/course/ultimate-agentic-ai-devops-with-claude-code/  
- 🎓 DevOps with Claude Code: Terraform, EKS, ArgoCD & Helm: https://www.udemy.com/course/devops-with-claude-code-terraform-eks-argocd-helm/  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*