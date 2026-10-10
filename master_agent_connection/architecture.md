# 🚀 Jenkins Master-Agent Setup Using Docker and SSH

![Docker](https://img.shields.io/badge/Docker-Containerization-2496ED?logo=docker&logoColor=white)
![Jenkins](https://img.shields.io/badge/Jenkins-CI%2FCD-D24939?logo=jenkins&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04-E95420?logo=ubuntu&logoColor=white)
![SSH](https://img.shields.io/badge/SSH-Key%20Authentication-4D4D4D?logo=openssh&logoColor=white)

A hands-on DevOps practice demonstrating how to set up a Jenkins controller and agent using Docker containers and SSH key-based authentication.

## 📑 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Technologies Used](#-technologies-used)
- [Setup Guide](#-setup-guide)
- [SSH Key Authentication](#-ssh-key-authentication)
- [Jenkins Agent Configuration](#-jenkins-agent-configuration)
- [Test the Connection](#-test-the-connection)
- [Troubleshooting](#-troubleshooting)
- [What I Learned](#-what-i-learned)

## 📌 Overview

The goal of this project is to configure a Jenkins controller and a separate Jenkins agent in Docker.

- **Jenkins Controller:** Manages jobs, credentials, and build scheduling.
- **Jenkins Agent:** Executes build commands assigned by the controller.
- **SSH:** Provides secure communication and key-based authentication.
- **Docker Network:** Allows containers to communicate with each other.

## 🏗️ Architecture

```text
                    Host Machine
                          |
                    Docker Network
                     jenkins-net
                          |
             +------------+------------+
             |                         |
      Jenkins Controller          Jenkins Agent
          Container                  Container
          `jenkins`                   `agent`
             |                         |
        SSH Private Key  -------->  Public Key
             |                      authorized_keys
             |                         |
             +------ SSH Connection ---+
                          |
                   Build Execution
```

## 🛠️ Technologies Used

| Technology | Purpose |
|---|---|
| Docker | Containerization |
| Jenkins | CI/CD automation |
| Ubuntu | Agent operating system |
| OpenSSH | Remote authentication |
| Java JDK | Running the Jenkins agent |

## ⚙️ Setup Guide

### 1. Create a Docker Network

```bash
docker network create jenkins-net
```

### 2. Create the Jenkins Agent

```bash
docker run -dit --name agent network jenkins-net ubuntu:24.04 bash
```

### 3. Install Java and OpenSSH

```bash
docker exec agent bash -c "apt update && apt install -y openjdk-21-jdk openssh-server"
```

### 4. Start the SSH Server

```bash
docker exec agent bash -c "mkdir -p /run/sshd && /usr/sbin/sshd"
```

Verify that SSH is running:

```bash
docker exec agent bash -c "ps aux | grep '[s]shd'"
```

### 5. Generate SSH Keys on the Controller

```bash
docker exec jenkins bash -c  "mkdir -p /var/jenkins_home/.ssh && ssh-keygen -t ed25519 -N ''  -f /var/jenkins_home/.ssh/id_ed25519"
```

> **Security:** Never publish your private key in a GitHub repository.

### 6. Copy the Public Key to the Agent

```bash
docker exec jenkins cat /var/jenkins_home/.ssh/id_ed25519.pub | docker exec -i agent bash -c "mkdir -p /root/.ssh && cat > /root/.ssh/authorized_keys && chmod 700 /root/.ssh && chmod 600 /root/.ssh/authorized_keys"
```

### 7. Test SSH Connectivity

```bash
docker exec jenkins ssh -i /var/jenkins_home/.ssh/id_ed25519 root@agent "java -version"
```

A successful connection should display the Java version installed on the agent.

## 🔑 SSH Key Authentication

SSH uses a public and private key pair.

| Key | Location | Purpose |
|---|---|---|
| Private key | Jenkins controller / Jenkins Credentials | Authenticates the controller |
| Public key | Agent's `authorized_keys` | Authorizes the controller |

### Connection Flow

1. Generate an SSH key pair on the controller.
2. Add the public key to the agent's `authorized_keys`.
3. Store the private key in Jenkins Credentials.
4. Configure the agent to launch through SSH.
5. Verify that the node becomes **Online**.



## 🔐 OpenSSH Server (sshd) Configuration

Installed and configured OpenSSH Server inside the Jenkins agent container.

Learned to start and verify the sshd daemon.

Explored sshd_config and SSH public-key authentication.

Enabled SSH connectivity between the Jenkins controller and agent.

### Start SSH server
docker exec agent bash -c "mkdir -p /run/sshd && /usr/sbin/sshd"

### Verify SSH daemon
docker exec agent bash -c "ps aux | grep '[s]shd'"




## 🖥️ Jenkins Agent Configuration

In Jenkins, navigate to **Manage Jenkins → Nodes** and configure a permanent agent.

| Setting | Example |
|---|---|
| Node name | `agent1` |
| Host | `agent` |
| Remote root directory | `/root/jenkins-agent` |
| Launch method | Launch agents via SSH |
| Credentials | SSH Username with private key |
| Labels | `docker-agent` |

The username and remote directory must match the account and permissions used on your agent.

## 🧪 Test the Agent with a Pipeline

Create a Jenkins Pipeline job using the following example:

```groovy
pipeline {
    agent {
        label 'docker-agent'
    }

    stages {
        stage('Verify Agent') {
            steps {
                sh 'hostname'
                sh 'java -version'
                echo 'Jenkins agent is working!'
            }
        }
    }
}
```

This pipeline runs on an agent with the `docker-agent` label.

## 🛠️ Troubleshooting

| Issue | What to check |
|---|---|
| Agent is offline | Verify SSH server and Docker network |
| Permission denied | Check the username and SSH keys |
| Credentials not found | Verify the credential ID and scope |
| Java not found | Install Java inside the agent container |
| Host key verification failed | Verify and configure the SSH host key |
| SSH server not running | Start `sshd` inside the agent |

## 📚 What I Learned

- Creating and connecting Docker containers.
- Understanding Jenkins controller-agent architecture.
- Installing Java and configuring OpenSSH.
- Generating and using SSH key pairs.
- Managing Jenkins SSH credentials.
- Configuring Jenkins agents through SSH.
- Troubleshooting authentication and connectivity issues.

## ✅ Final Result

The Jenkins controller can connect to the agent through SSH, and Jenkins jobs can run on the agent when the job configuration matches its label.

---

**Project:** Jenkins Master-Agent Setup  
**Focus:** Docker · Jenkins · SSH · CI/CD Automation
