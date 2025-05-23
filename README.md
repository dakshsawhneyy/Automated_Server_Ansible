# Automated Web Server Deployment with Ansible on AWS

This project automates the entire lifecycle of a secure web server on AWS EC2 using **Ansible**. It provisions infrastructure, configures users and firewalls, installs a web server, deploys a motivational HTML site, sets up cron jobs, secures secrets with Ansible Vault, and finally allows teardown — all hands-free.

---

## Features

- 🟣 Provision EC2 using `amazon.aws.ec2_instance`
- 🟣 Secure SSH access with user creation and key auth
- 🟣 Harden server with `ufw` firewall (port 80 open)
- 🟣 Deploy a motivational static site using NGINX
- 🟣 Schedule cron job for server tasks
- 🟣 Secure AWS secrets via `ansible-vault`
- 🟣 Destroy instance cleanly with Ansible
- 🟣 Modular role-based structure

---

## 📁 Project Structure

```bash
Automated-Web-Server-Deployment/
├── inventory.ini
├── vault.pass
├── vault.yml
├── role.yml
├── create_ec2.yml
├── destroy_ec2.yml
├── files/
│   └── index.html
├── roles/
│   ├── user/
│   │   └── tasks/main.yml
│   ├── firewall/
│   │   └── tasks/main.yml
│   ├── webserver/
│   │   └── tasks/main.yml
│   └── cron/
│       └── tasks/main.yml
└── README.md
```

---

## 🔐 Setting Up Vault Secrets

1. Create the vault.yml file:

```bash
ansible-vault create vault.yml
```

2. Add AWS credentials securely:

```yaml
aws_access_key: YOUR_ACCESS_KEY
aws_secret_key: YOUR_SECRET_KEY
```

3. Create the vault.pass file and store your vault password in it:

```text
your-vault-password
```

 Add `vault.pass` to your `.gitignore`.

---

## 📝 inventory.ini Example

```ini
[all]
ubuntu ansible_host=YOUR_INSTANCE_PUBLIC_IP ansible_user=ubuntu

[all:vars]
ansible_ssh_private_key_file=/path/to/your-key.pem
```

---

## 🧾 Usage

### Create EC2 Instance

```bash
ansible-playbook create_ec2.yml --vault-password-file vault.pass
```

### Configure Server + Deploy Webpage

```bash
ansible-playbook role.yml --inventory inventory.ini --vault-password-file vault.pass
```

### Destroy EC2 Instance

```bash
ansible-playbook destroy_ec2.yml --vault-password-file vault.pass
```

---

## 🔵role.yml (Main Playbook)

```yaml
- name: Configure Web Server
  hosts: all
  become: true

  roles:
    - user
    - firewall
    - webserver
    - cron
```

---

## 🔵 create_ec2.yml

```yaml
- name: Launch EC2 instance
  hosts: localhost
  connection: local
  gather_facts: false

  vars_files:
    - vault.yml

  tasks:
    - name: Launch instance
      amazon.aws.ec2_instance:
        key_name: general-key-pair
        instance_type: t2.micro
        image_id: ami-xxxxxxxxxxxxx
        region: ap-south-1
        wait: true
        count: 1
        network:
          assign_public_ip: true
        security_group: default
        tags:
          Name: ansible-instance
        aws_access_key: "{{ aws_access_key }}"
        aws_secret_key: "{{ aws_secret_key }}"
```

---

## 🔨 destroy_ec2.yml

```yaml
- name: Terminate EC2 Instance
  hosts: localhost
  connection: local
  gather_facts: false

  vars_files:
    - vault.yml

  tasks:
    - name: Terminate instances
      amazon.aws.ec2_instance:
        state: absent
        filters:
          tag:Name: ansible-instance
        aws_access_key: "{{ aws_access_key }}"
        aws_secret_key: "{{ aws_secret_key }}"
        region: ap-south-1
```

---

## 👤 User Role (roles/user/tasks/main.yml)

```yaml
- name: Create deploy user
  user:
    name: deploy
    shell: /bin/bash
    create_home: yes

- name: Create .ssh dir
  file:
    path: /home/deploy/.ssh
    state: directory
    mode: '0700'
    owner: deploy
    group: deploy

- name: Add public key
  authorized_key:
    user: deploy
    state: present
    key: "{{ lookup('file', 'files/deploy_key.pub') }}"
```

---

## 🔵 Firewall Role (roles/firewall/tasks/main.yml)

```yaml
- name: Install UFW
  apt:
    name: ufw
    state: present
    update_cache: yes

- name: Allow port 80
  command: ufw allow 'Nginx HTTP'

- name: Enable UFW
  command: ufw --force enable
```

---

## 🌐 Webserver Role (roles/webserver/tasks/main.yml)

```yaml
- name: Install nginx
  apt:
    name: nginx
    state: present
    update_cache: yes

- name: Copy index.html
  copy:
    src: files/index.html
    dest: /var/www/html/index.html
    owner: www-data
    group: www-data
    mode: '0644'

- name: Enable nginx
  service:
    name: nginx
    state: started
    enabled: true
```

---

## ⏰ Cron Role (roles/cron/tasks/main.yml)

```yaml
- name: Add daily cron cleanup job
  cron:
    name: "Clean /tmp files"
    minute: "0"
    hour: "3"
    job: "/usr/bin/find /tmp -type f -atime +10 -delete"
    user: deploy
```

---

## Requirements

Python packages:

```bash
pip install boto3 botocore ansible amazon.aws
```

Correct SSH permissions:

```bash
chmod 600 /path/to/your-key.pem
```

Ansible version 2.10+ recommended

---

## 🙌 Final Note

This project is your stepping stone to professional-grade Infrastructure as Code.
You're automating from zero to production — EC2 provisioning, firewalling, web server setup, scheduled tasks, and teardown.
Keep this grind alive, and you're headed straight for elite DevOps roles.

**"Make your systems work for you — while you sleep."**
