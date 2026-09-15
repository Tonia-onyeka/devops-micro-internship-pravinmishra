# Assignment 04 — Deploy Mini Finance on Azure Using Terraform and Ansible

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

In this assignment, you will provision Azure infrastructure using Terraform and deploy the Mini Finance website using an Ansible multi-play playbook.

Terraform will create the Azure Virtual Machine and networking resources. Ansible will install Nginx, clone the Mini Finance repository, deploy the website, and verify the deployment.

---

# Task 1 — Create the Project Structure

## Goal

Create separate directories and files for the Terraform infrastructure and Ansible configuration.

### Evidence

#### Screenshot 1 — Terminal or VS Code showing the complete `mini-finance` project structure

![Screenshot](screenshots/Ass4.Task1.ss1.png)

---

### Notes

Set up the Mini-Finance project structure, Git configuration, .gitignore, README, and required Terraform and Ansible directories. Verified the development environment and prepared the workspace for infrastructure automation.

---

# Task 2 — Create the Azure Infrastructure Using Terraform

## Goal

Use Terraform to provision an Ubuntu Virtual Machine with the required Azure networking and security resources.

### Evidence

#### Screenshot 2 — Terraform code showing the `Allow-SSH` rule for port `22` and the `Allow-HTTP` rule for port `80`

![Screenshot](screenshots/Ass4.Task2.ss2.png)

---

#### Screenshot 3 — Terraform code showing the association between `nsg-mini-finance` and `nic-mini-finance`

![Screenshot](screenshots/Ass4.Task2.ss3.png)

---

### Notes

Created the Terraform configuration for the Azure infrastructure, including the resource group, virtual network, subnet, network security group, public IP, network interface, and Linux virtual machine. Configured SSH access using an existing Ed25519 public key and restricted SSH access to the current public IP.

---

# Task 3 — Initialize and Apply the Terraform Configuration

## Goal

Format and validate the Terraform configuration, review the execution plan, and provision the Azure infrastructure.

### Evidence

#### Screenshot 4 — End of the `terraform apply` output showing `Apply complete!` with no errors

![Screenshot](screenshots/Ass4.Task3.ss4.png)

---

#### Screenshot 5 — Output of `terraform output public_ip` showing the VM’s public IP address

![Screenshot](screenshots/Ass4.Task3.ss5.png)

---

### Notes

Initialized, formatted, validated, planned, and applied the Terraform configuration successfully. After resolving Azure regional and VM SKU capacity constraints, the infrastructure was deployed successfully in West US 2. The deployment completed with 8 resources created and 0 changes or destructions, and the VM received a public IP address for the upcoming Ansible configuration.

---

# Task 4 — Verify Passwordless SSH Access

## Goal

Confirm that the Ansible controller can connect to the Terraform-provisioned Azure VM using SSH key authentication.

### Evidence

#### Screenshot 6 — Passwordless SSH command and the returned `mini-finance` hostname

![Screenshot](screenshots/Ass4.Task4.ss6.png)

---

### Notes

Verified passwordless SSH access from the Ansible controller to the Terraform-provisioned Azure VM using the Ed25519 SSH key. The connection succeeded without requesting an azureuser password, and the returned hostname confirmed the target VM as mini-finance.

---

# Task 5 — Create the Ansible Inventory and Verify Connectivity

## Goal

Add the Terraform-provisioned Azure VM to the Ansible inventory and confirm that Ansible can connect to it.

### Evidence

#### Screenshot 7 — Ansible ping output showing `SUCCESS` and `pong` from the Azure VM

![Screenshot](screenshots/Ass4.Task5.ss7.png)

---

### Configuration File

Copy and paste the complete contents of your `ansible/inventory.ini` file below:

```ini
[web]
20.3.152.243

[web:vars]
ansible_user=azureuser
ansible_ssh_private_key_file=~/.ssh/id_ed25519

```

---

# Task 6 — Create the Multi-Play Ansible Playbook

## Goal

Create one Ansible playbook containing separate plays to install Nginx, deploy the Mini Finance website, and verify the deployment.

### Evidence

#### Screenshot 8 — `site.yml` showing Play 1 and the beginning of Play 2

Screenshot must show:

- Play 1 targeting the `web` group
- Installation of `nginx`, `git`, and `rsync`
- Nginx service configured as started and enabled
- Beginning of Play 2 with the Git repository URL and synchronization task

![Screenshot](screenshots/Ass4.Task6.ss8.png)

---

#### Screenshot 9 — `site.yml` showing the deployment destination, handler, and Play 3 verification

Screenshot must show:

- Website destination `/var/www/html/`
- Ownership set to `www-data:www-data`
- Nginx reload handler
- Play 3 targeting `localhost`
- The `uri` verification and `assert` condition

![Screenshot](screenshots/Ass4.Task6.ss9.png)

---

### Configuration File

Copy and paste the complete contents of your `ansible/site.yml` file below:

```yaml
---
- name: Install and configure Nginx
  hosts: web
  become: true

  tasks:
    - name: Update apt package cache
      ansible.builtin.apt:
        update_cache: true

    - name: Install required packages
      ansible.builtin.apt:
        name:
          - nginx
          - git
          - rsync
        state: present

    - name: Ensure Nginx is started and enabled
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true


- name: Clone and deploy Mini Finance website
  hosts: web
  become: true

  tasks:
    - name: Clone or update Mini Finance repository
      ansible.builtin.git:
        repo: "https://github.com/pravinmishraaws/mini-finance-project"
        dest: /opt/mini-finance
        version: main
        update: true

    - name: Synchronize website files to Nginx web root
      ansible.posix.synchronize:
        src: /opt/mini-finance/
        dest: /var/www/html/
        delete: true
        rsync_opts:
          - "--exclude=.git"
      delegate_to: "{{ inventory_hostname }}"
      notify: Reload Nginx

    - name: Set ownership of website files
      ansible.builtin.file:
        path: /var/www/html/
        owner: www-data
        group: www-data
        recurse: true

  handlers:
    - name: Reload Nginx
      ansible.builtin.service:
        name: nginx
        state: reloaded


- name: Verify Mini Finance deployment
  hosts: localhost
  connection: local
  become: false

  tasks:
    - name: Request Mini Finance website
      ansible.builtin.uri:
        url: "http://{{ hostvars[groups['web'][0]]['ansible_host'] | default(groups['web'][0]) }}"
        method: GET
        status_code: 200
      register: website_response

    - name: Verify HTTP response
      ansible.builtin.assert:
        that:
          - website_response.status == 200
        success_msg: "Mini Finance website is responding with HTTP 200."
        fail_msg: "Mini Finance website did not return HTTP 200."

```

---

# Task 7 — Validate and Run the Ansible Playbook

## Goal

Validate the syntax of the multi-play Ansible playbook and run it to install Nginx, deploy the Mini Finance website, and verify the deployment.

### Evidence

#### Screenshot 10 — Successful playbook syntax check showing `playbook: site.yml`

![Screenshot](screenshots/Ass4.Task7.ss10.png)

---

#### Screenshot 11 — Play 3 output showing the successful HTTP verification and assertion

![Screenshot](screenshots/Ass4.Task7.ss11.png)

---

#### Screenshot 12 — Final `PLAY RECAP` showing `failed=0` and `unreachable=0`

![Screenshot](screenshots/Ass4.Task7.ss12.png)

---

### Notes

The Ansible playbook was successfully validated using the syntax check, confirming that `site.yml` contains valid Ansible syntax. The playbook was then executed successfully to install and configure Nginx, clone and deploy the Mini Finance website, synchronize the website files to the Nginx web root, and reload Nginx.

The final verification play successfully confirmed that the deployed website returned HTTP 200. The final PLAY RECAP showed `failed=0` and `unreachable=0`, confirming that all tasks completed successfully without any failed or unreachable hosts.


---

# Task 8 — Test the Mini Finance Website in a Browser

## Goal

Confirm that the Mini Finance website is publicly accessible through the Azure VM’s public IP address.

### Evidence

#### Screenshot 13 — Mini Finance website successfully loading in the browser, with the Azure VM’s public IP address visible in the address bar

![Screenshot](screenshots/Ass4.Task48.ss13.png)

---

### Website URL

Add your deployed website URL below:

http://20.3.152.243/


---

# Task 9 — Create the Project README

## Goal

Create a `README.md` file to document the Mini Finance infrastructure and deployment project.

### Evidence

#### Screenshot 14 — Completed `README.md` displayed in the VS Code Markdown preview or terminal

![Screenshot](screenshots/Ass4.Task9.ss14.png)

---

### README Content

Copy and paste the complete contents of your `README.md` file below:

```markdown
# Mini Finance Website Deployment with Terraform and Ansible

## Project Objective

This project demonstrates how to provision cloud infrastructure with Terraform on Microsoft Azure and use Ansible to configure the server and deploy a static Mini Finance website.

The deployment includes an Azure Ubuntu virtual machine running Nginx. Ansible automates the installation and configuration of Nginx, deployment of the Mini Finance website, and verification that the website is responding successfully.

## Tools and Technologies

* **Terraform** — Infrastructure as Code for provisioning Azure resources
* **Microsoft Azure** — Cloud platform hosting the infrastructure
* **Ansible** — Server configuration and application deployment automation
* **Nginx** — Web server used to serve the Mini Finance website
* **Git** — Version control for the project files
* **rsync** — Synchronization of website files to the Nginx web root

## Infrastructure Created

The Terraform configuration provisions the following Azure resources:

* **Resource Group** — Contains the project resources
* **Virtual Network** — Provides private network connectivity
* **Subnet** — Provides a network segment for the virtual machine
* **Network Security Group** — Controls inbound and outbound network traffic
* **Public IP Address** — Provides public access to the virtual machine
* **Network Interface** — Connects the virtual machine to the Azure network
* **Ubuntu Virtual Machine** — Hosts the Nginx web server and Mini Finance website

## Ansible Deployment Workflow

The Ansible playbook uses multiple plays to automate the deployment process.

### 1. Install and Configure Nginx

Ansible updates the package cache, installs the required packages, ensures that Nginx is running, and enables the service.

### 2. Clone and Deploy the Mini Finance Website

Ansible clones or updates the Mini Finance project repository and synchronizes the website files to the Nginx web root using `rsync`.

The website files are then assigned the appropriate ownership and permissions, and Nginx is reloaded.

### 3. Verify the Deployment

Ansible sends an HTTP request to the deployed website and verifies that the server returns HTTP status code `200`.

A successful verification confirms that the website is being served correctly by Nginx.

## Verification

The deployment was verified in two ways.

First, the Ansible playbook successfully completed all deployment tasks and returned the following verification message:

`Mini Finance website is responding with HTTP 200.`

The final Ansible recap also confirmed that there were no failed or unreachable hosts.

Second, the website was opened in a web browser using the Azure virtual machine's public IP address:

`http://<PUBLIC_IP>`

The Mini Finance website loaded successfully, confirming that Nginx was publicly serving the deployed website.

## Challenge and Solution

One important point checked during the deployment was ensuring that the Azure virtual machine was reachable through its public IP address and that HTTP traffic was permitted by the Network Security Group.

The deployment was verified by confirming that the required network access was available, Nginx was running, and the Ansible HTTP verification returned status code `200`.

## What I Learned

This project helped me understand how Terraform and Ansible can work together in a DevOps workflow.

Terraform was used to provision and manage the Azure infrastructure as code, while Ansible was used to configure the provisioned virtual machine and automate the website deployment.

I also learned how to use Ansible multi-play playbooks, inventory files, handlers, package management, Git repository deployment, `rsync`, and automated HTTP verification.

Overall, the project demonstrated how infrastructure provisioning and server configuration can be automated to create a repeatable and reliable deployment process.

```

---

# LinkedIn Post Required

## Evidence

#### Screenshot 15 — Published LinkedIn post showing the text and at least one deployment screenshot

![Screenshot](screenshots/Ass4.LinkedInPost.png)

---

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

https://www.linkedin.com/posts/anthonia-akwuohia-5b00681b0_devops-terraform-ansible-share-7504968470585987072-mIVq/?utm_source=share&utm_medium=member_desktop&rcm=ACoAADEhX1QBTHiW-kQPmKjn3MVixQzj4IzJO1Q

---

### LinkedIn Submission Notes

**One challenge you faced and how you fixed it:**

One challenge was ensuring that the Azure VM was accessible over HTTP and that Nginx was correctly serving the deployed website. I resolved this by checking the Azure Network Security Group rules, confirming that HTTP port 80 was allowed, verifying that Nginx was running, and using Ansible to confirm that the website returned HTTP status code 200.

---

**One real-world example where you can use this learning:**

This approach can be used in a real-world web deployment environment where Terraform provisions cloud infrastructure and Ansible automatically configures servers and deploys applications. For example, a company could use Terraform to create Azure web servers and Ansible to install Nginx and deploy a website consistently across multiple environments.

---

# Assignment Questions

Answer the following in your own words:

**1. What did you provision using Terraform in this assignment?**

I used Terraform to provision the Azure infrastructure required to host the Mini Finance website. This included a Resource Group, Virtual Network, Subnet, Network Security Group, Public IP address, Network Interface, and Ubuntu Virtual Machine.

---

**2. What did Ansible configure and deploy in this assignment?**

Ansible configured the Ubuntu virtual machine by installing and starting Nginx. It also cloned the Mini Finance website repository, synchronized the website files to the Nginx web root, set the appropriate file ownership, and reloaded Nginx.

---

**3. Why is SSH access on port `22` restricted to your public IP address?**

SSH port 22 is restricted to my public IP address to reduce the risk of unauthorized access. This means that only my trusted network can connect to the server through SSH instead of exposing SSH access to the entire internet.

---

**4. Why is HTTP port `80` open to the internet?**

HTTP port 80 is open because the Mini Finance website needs to be publicly accessible through a web browser. Allowing inbound traffic on port 80 enables users to access the website using HTTP.

---

**5. What is the purpose of the Ansible inventory file?**

The Ansible inventory file identifies the managed servers and provides the connection information Ansible needs to communicate with them. In this assignment, it identifies the Azure VM that Ansible configures and deploys the website to.
---

**6. Why does the playbook use separate plays for install, deploy, and verify?**

Separate plays make the playbook easier to understand, manage, and troubleshoot. The first play prepares the server, the second deploys the website, and the third verifies that the deployment was successful. This also creates a clear and logical deployment workflow.

---

**7. Why is `rsync` useful when deploying website files?**

rsync is useful because it efficiently synchronizes files between locations and only transfers files that have changed. This makes website deployments faster and avoids unnecessarily copying the entire website every time.

---

**8. What does the Ansible `uri` module verify in this assignment?**

The Ansible uri module sends an HTTP request to the deployed website and checks the response. In this assignment, it verifies that the website is accessible and returns the expected HTTP status code 200.

---

**9. What issue did you face during this assignment, and how did you fix it?**

One issue I encountered was ensuring that the deployed website was accessible through the Azure VM's public IP address. I checked the Network Security Group configuration, confirmed that HTTP port 80 was allowed, verified that Nginx was running, and then used Ansible's HTTP verification to confirm that the website returned HTTP 200.

---

**10. What did you learn from using Terraform and Ansible together?**

I learned that Terraform and Ansible can complement each other in a DevOps workflow. Terraform can be used to provision the cloud infrastructure as code, while Ansible can configure the provisioned servers and automate application deployment. Using both tools makes the deployment process more repeatable, consistent, and easier to manage.
---

# Required Files

Confirm that the following files are included in your assignment folder:

- [ ] `.gitignore`
- [ ] `README.md`
- [ ] `terraform/providers.tf`
- [ ] `terraform/main.tf`
- [ ] `terraform/variables.tf`
- [ ] `terraform/outputs.tf`
- [ ] `ansible/inventory.ini`
- [ ] `ansible/site.yml`

---

# Submission Instructions

- Add all required screenshots in the correct order.
- Full Name must be visible in required screenshots.
- Add the Azure VM public IP address.
- Add the final Mini Finance website URL.
- Paste `inventory.ini`, `site.yml`, and `README.md` as editable text.
- Answer all assignment questions clearly in your own words.
- Add your LinkedIn post URL.
- Do not expose SSH private keys, passwords, Azure credentials, subscription IDs, Terraform state contents, or other sensitive information.

---

# Completion Checklist

- [ ] Task 1: `mini-finance` project structure created
- [ ] Task 1: `.gitignore` created
- [ ] Task 2: Terraform Azure infrastructure code created
- [ ] Task 2: `Allow-SSH` rule configured for port `22`
- [ ] Task 2: `Allow-HTTP` rule configured for port `80`
- [ ] Task 2: NSG associated with the Network Interface
- [ ] Task 3: `terraform fmt` completed
- [ ] Task 3: `terraform init` completed
- [ ] Task 3: `terraform validate` completed successfully
- [ ] Task 3: `terraform apply` completed successfully
- [ ] Task 3: `terraform output public_ip` displayed the VM public IP
- [ ] Task 4: Passwordless SSH works from the Ansible controller
- [ ] Task 5: `inventory.ini` created
- [ ] Task 5: Ansible ping returns `SUCCESS` and `pong`
- [ ] Task 6: `site.yml` contains three separate plays
- [ ] Task 6: Play 1 installs Nginx, Git, and rsync
- [ ] Task 6: Play 2 clones and deploys the Mini Finance website
- [ ] Task 6: Play 3 verifies HTTP status code `200`
- [ ] Task 7: Playbook syntax check passes
- [ ] Task 7: Ansible playbook completes successfully
- [ ] Task 7: Final recap shows `failed=0` and `unreachable=0`
- [ ] Task 8: Mini Finance website loads in the browser
- [ ] Task 8: Azure VM public IP is visible in the browser screenshot
- [ ] Task 9: `README.md` completed
- [ ] Screenshots 1–15 are included
- [ ] `inventory.ini`, `site.yml`, and `README.md` are pasted as editable text
- [ ] Assignment questions are answered
- [ ] LinkedIn post published with Anyone visibility
- [ ] LinkedIn post URL added
- [ ] No sensitive information is exposed

---

## About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra and The CloudAdvisory, focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## Resources

- DMI Official Website: [https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme](https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme)
- University: [https://university.pravinmishra.com?utm_source=github&utm_medium=readme](https://university.pravinmishra.com?utm_source=github&utm_medium=readme)
- Discord Community: [https://discord.pravinmishra.com?utm_source=github&utm_medium=readme](https://discord.pravinmishra.com?utm_source=github&utm_medium=readme)
- Blog: [https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme](https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme)
- YouTube Playlist: [https://www.youtube.com/playlist?list=PLFeSNDtI4Cho](https://www.youtube.com/playlist?list=PLFeSNDtI4Cho)
- Pravin Mishra LinkedIn: [https://www.linkedin.com/in/pravin-mishra-aws-trainer/](https://www.linkedin.com/in/pravin-mishra-aws-trainer/)
- CloudAdvisory LinkedIn: [https://www.linkedin.com/company/thecloudadvisory/](https://www.linkedin.com/company/thecloudadvisory/)

---

*This submission is part of DevOps Micro Internship (DMI) — Agentic AI Track.*