# Infrastructure Quest: Automated Web Stack Deployment (Ansible x Nginx x AWS)

This project demonstrates a fully automated, idempotent approach to deploying a static web server stack on Amazon Web Services (AWS) using **Ansible** as the primary Configuration Management tool. The entire setup adheres strictly to **Infrastructure as Code (IaC)** principles.

The goal is to move an EC2 instance from its initial, raw state to a fully configured Nginx web server, serving a custom, styled landing page—all executed via a single command from an Ansible Control Node.

**Architected and Coded **

## Install Ansible AMI LINUX 2023
``´bash
sudo dnf install python3-pip
pip install ansible
``´

##  Project Goals

* **Implement IaC:** Define infrastructure configuration in code (`setup_nginx.yml`).

* **Achieve Idempotency:** Ensure the playbook can be run repeatedly without causing unintended side effects.

* **Automate Deployment:** Handle OS updates, package installation (Nginx), service management, and content deployment automatically.

* **Deliver Modern UI:** Deploy a custom, professionally styled HTML landing page (dark mode, gamified success message).

##  Architecture Overview

The system consists of two primary components running on AWS EC2 instances:

| **Component** | **Role** | **Function** |
| :--- | :--- | :--- |
| **Ansible Control Node** | Orchestrator | Stores the playbook (`setup_nginx.yml`) and inventory (`inventory.ini`). Executes the `ansible-playbook` command. |
| **Web Server (Target)** | Managed Node | The EC2 instance where Ansible connects via SSH to install and configure **Nginx** and deploy the web content. |

##  Prerequisites

Before running the playbook, ensure you have the following in place:

1. **AWS Account:** Two running EC2 instances (one Ubuntu for the Control Node, one for the Web Server).

2. **Security Groups:** Ensure the Web Server's security group allows:

   * Inbound SSH (Port 22) from the Control Node's IP address.

   * Inbound HTTP (Port 80) from `0.0.0.0/0` (for public access).

3. **SSH Key:** The private key (`Ansible-project.pem`) used to launch and access both instances must be available on the **Control Node** and secured with correct permissions (`chmod 400 Ansible-project.pem`).

4. **Ansible:** Must be installed on the Control Node.

##  Configuration & Usage

### 1. Configure the Inventory

The `inventory.ini` file defines the target server's connection details. Ensure the `ansible_host` value is updated with the **Public DNS** or **Public IP** of your AWS Web Server instance, and that the `ansible_user` matches the correct image login (e.g., `ubuntu`).

```

[webservers]

# Replace the hostname with your target EC2 instance's Public DNS/IP

web1 ansible\_host=\<YOUR\_EC2\_PUBLIC\_DNS\> ansible\_user=ubuntu

[all:vars]

# Ensure this file exists and has 400 permissions

ansible\_ssh\_private\_key\_file=Ansible-project.pem

```

### 2. File Structure

Organize the project files on the Ansible Control Node as follows:

```

.
├── inventory.ini
├── setup\_nginx.yml
└── site/
└── index.html         \# Custom styled landing page

```

### 3. Run the Deployment

From the root directory of the **Ansible Control Node**, execute the playbook:

```

# This command runs the automation against the hosts defined in inventory.ini

ansible-playbook -i inventory.ini setup\_nginx.yml

```

### 4. Verify Success

After the playbook completes successfully (check that `unreachable=0` and `changed > 0`), navigate to the Web Server's Public IP address in your browser:

```

http://\<YOUR\_EC2\_PUBLIC\_IP\>

```

You should see the custom, gamified "Deployment Success!" landing page.

##  The Ansible Playbook (`setup_nginx.yml`)

The playbook is simple, ensuring dependency installation and content deployment in three steps:

| **Task Name** | **Module Used** | **Description** |
| :--- | :--- | :--- |
| **1. Ensure Nginx is installed** | `ansible.builtin.apt` | Installs the latest version of Nginx. |
| **2. Ensure Nginx service is running** | `ansible.builtin.service` | Starts the Nginx service and ensures it restarts on reboot (`enabled: yes`). |
| **3. Deploy custom index.html file** | `ansible.builtin.copy` | Transfers the styled HTML content from the Control Node to the Nginx default root directory (`/var/www/html/`). |

![deployment](https://github.com/user-attachments/assets/379bc18f-baf2-4a6f-9d00-76a136f7f25d)




##  Key Takeaways

* **Idempotency:** The playbook ensures Nginx is *present* and *started*, preventing unnecessary actions if the server is already configured.

* **Separation of Concerns:** The application logic (HTML/CSS) is separate from the infrastructure logic (Ansible YAML).

* **Scalability:** To deploy 10 web servers instead of 1, only the `inventory.ini` file needs to be updated.

If you have feedback on the playbook or architecture, feel free to reach out or contribute!
