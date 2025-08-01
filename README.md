# n8n-docker-swarm-setup
Docker Swarm deployment setup for n8n with PostgreSQL backend – includes local and Swarm mode configurations.

**n8n Docker Compose Setup – Phase 1: Local Test on Ubuntu EC2**
This repository provides step-by-step instructions to set up the n8n workflow automation platform using Docker Compose on a single Ubuntu EC2 instance. It includes integration with PostgreSQL and testing connectivity and functionality via Docker.

**Project Overview**
Objective: Test the docker-compose setup of n8n + PostgreSQL locally on an Ubuntu EC2 instance.

**Components:**
n8n (latest)
PostgreSQL (16-alpine)
Setup Type: Single-host Docker Compose deployment
Author: Mahendra Kumar

**Target Reviewer:** Ajay Nair – VP, Enterprise Architecture & Integration Solutions (ajayn@unitedtechno.com)

**Repository Contents**
docker-compose.yml – Main Docker Compose configuration file

.env – Environment variables for PostgreSQL and n8n (not committed for security)

README.md – Setup instructions

**Prerequisites**
AWS EC2 instance (Ubuntu)
.pem key file for SSH access
PuTTY and PuTTYgen (for .ppk conversion)
Docker and Docker Compose installed

**Setup Instructions**
Step 1: Launch EC2 (Ubuntu)
Create a new EC2 instance with Ubuntu using the AWS Free Tier.

Step 2: Convert PEM to PPK
Use PuTTYgen to convert .pem to .ppk.

Step 3: SSH into EC2 via PuTTY
Use the .ppk private key to log into your Ubuntu server.

Step 4: Install Docker & Docker Compose
sudo apt update && sudo apt upgrade -y
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

# Add Docker GPG Key and Repository
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] \
https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io

# Install Docker Compose (v2.24.2)
sudo curl -L "https://github.com/docker/compose/releases/download/v2.24.2/docker-compose-$(uname -s)-$(uname -m)" \
-o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
docker --version

Step 5: Add Docker Permissions
sudo usermod -aG docker $USER
newgrp docker

Step 6: Transfer Compose File to Ubuntu
From Windows PowerShell:
scp -i .\N8N_KeyPair.pem .\docker-compose.yml ubuntu@<EC2-Public-IP>:/home/ubuntu

Step 7: Prepare Environment
Create a .env file with the following content:
env
POSTGRES_USER=n8n
POSTGRES_PASSWORD=securepass
POSTGRES_DB=n8nd
N8N_ENCRYPTION_KEY=<your_random_key>
N8N_USER_MANAGEMENT_JWT_SECRET=<your_jwt_secret>

**Generate secure keys:**
openssl rand -hex 32

Step 8: Start Docker Compose
cd /home/ubuntu
docker-compose up -d

**Post-Deployment Checks**
Check Running Containers
docker ps

View Logs
docker logs <n8n_container_id>
docker logs <postgres_container_id>
Check Web UI
Visit:
http://<your-ec2-public-ip>:5678

**Troubleshooting**
Secure Cookie Error: Add this to the n8n environment in docker-compose.yml:
- N8N_SECURE_COOKIE=false
  
**Port Not Accessible:** Ensure port 5678 is open in your EC2 Security Group.

**Database Errors:**
If you see database "n8n" does not exist, verify the PostgreSQL container initialized correctly.

**Create a Sample Workflow**
Open n8n in browser.
Click “Create Workflow”.
Add a Manual Trigger node.
Add a Set node to output sample data.
Execute the workflow and verify via database:

docker exec -it <postgres_container_id> bash

psql -U n8n -d n8n
\dt
SELECT * FROM workflow_entity;
SELECT * FROM execution_entity ORDER BY id DESC LIMIT 5;

**Summary**
Docker Compose setup is working 
n8n connects to PostgreSQL 
Able to access UI and run workflows 
This completes Phase 1: Local Testing.

Next phase: Docker Swarm conversion.
