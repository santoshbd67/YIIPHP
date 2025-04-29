# DevOps Assessment Task – Yii2 + Docker Swarm + CI/CD + Ansible

### Created an Ec2 instance of t2.medium with help of terraform
#### Login to the Instance

# ✅ 1. Prerequisites on EC2 Ubuntu
### First, make sure your EC2 instance is ready:

#### OS: Ubuntu 20.04/22.04 (you can check: lsb_release -a)

#### You have ssh access with a keypair

#### You have sudo privileges (you can run sudo commands)

# ✅ 2. Install Ansible (on the same EC2 instance)
### If you already have Ansible installed, you can skip this step.

#### SSH into your EC2 instance:
```
ssh -i "your-key.pem" ubuntu@your-ec2-public-ip
```
#### Install Ansible:
```
sudo apt update
sudo apt install -y ansible
```
#### Confirm Ansible is installed:
```
ansible --version
```

# ✅ 3. Create Project Directory
### Create a working directory:
```
mkdir ~/ansible
cd ~/ansible
```

# ✅ 4. Create an Inventory File
### Create a file called inventory.ini:
```
[servers]
localhost ansible_connection=local ## you can use the ec2-public-Ip
```

#### Since you're running Ansible on the same machine, we use localhost and ansible_connection=local.

# ✅ 5. Create the Ansible Playbook
### Now, create a file called setup.yml.
```
nano setup.yml
```
#### Paste the following playbook:
```
---
- name: Infrastructure Setup with Ansible
  hosts: servers
  become: true

  tasks:
    - name: Install necessary system packages
      apt:
        name:
          - apt-transport-https
          - ca-certificates
          - curl
          - software-properties-common
          - gnupg-agent
          - git
          - nginx
          - php
          - php-fpm
        state: present
        update_cache: yes

    - name: Add Docker GPG key
      apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present

    - name: Add Docker APT repository
      apt_repository:
        repo: deb https://download.docker.com/linux/ubuntu {{ ansible_distribution_release }} stable
        state: present

    - name: Install Docker and Docker Compose
      apt:
        name:
          - docker-ce
          - docker-ce-cli
          - containerd.io
          - docker-compose-plugin
        state: present
        update_cache: yes

    - name: Enable and start Docker service
      systemd:
        name: docker
        state: started
        enabled: true

    - name: Initialize Docker Swarm
      shell: |
        docker swarm init
      args:
        creates: /tmp/swarm_initialized
      register: swarm_init_result
      changed_when: swarm_init_result.rc == 0

    - name: Create a marker file for Swarm Init
      file:
        path: /tmp/swarm_initialized
        state: touch
      when: swarm_init_result.rc == 0
```

# ✅ 6. Explanation of the Playbook

| Section | Purpose |
|:-------:|:-------:|
| Install packages | Install Git, NGINX, PHP, Docker prerequisites |
| Add Docker GPG Key and Repo | Install Docker the right way |
| Install Docker and Docker Compose | Latest versions |
| Enable/start Docker | Make Docker daemon persistent |
| Initialize Swarm | Run `docker swarm init` only if not already initialized |

# ✅ 7. Run the Playbook
### Now, execute the Ansible playbook:
```
ansible-playbook -i inventory.ini setup.yml
```
### You should see a step-by-step output where:

#### System packages are installed

#### Docker is installed

#### Swarm is initialized

#### NGINX, Git, PHP installed


---

# Application Deployment

I have manually created the source code for this application using Composer, which is available in this GitHub repository

# Application Deployment using GitHub Actions

I have created a `main.yml` file for the GitHub Action, which is located in the `.github/workflows/main.yml` directory. Below is the content of that file:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout Code
      uses: actions/checkout@v3
      with:
        fetch-depth: 1

    - name: Print Working Directory (debugging step)
      run: pwd

    - name: List Files (debugging step)
      run: ls -la

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2

    - name: Login to DockerHub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Build and Push Docker Image
      run: |
        docker build -f src/Dockerfile -t santoshbd67/yiiphp:latest src/
        docker push santoshbd67/yiiphp:latest

    - name: SSH into EC2 and update service
      uses: appleboy/ssh-action@v0.1.8
      with:
        host: ${{ secrets.EC2_PUBLIC_IP }}
        username: ubuntu
        key: ${{ secrets.EC2_SSH_KEY }}
        script: |
          docker pull santoshbd67/yiiphp:latest
          docker service update --image santoshbd67/yiiphp:latest yii2app
```


## Make sure to set the appropriate secrets in the GitHub repository settings:

### DOCKER_USERNAME

### DOCKER_PASSWORD

### EC2_PUBLIC_IP

### EC2_SSH_KEY








