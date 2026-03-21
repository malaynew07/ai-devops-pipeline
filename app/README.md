# 🤖 Agentic AI-Driven DevOps Pipeline
**Automated CI/CD with Intelligent Log Analysis**

![Status](https://img.shields.io/badge/Status-Success-brightgreen)
![Tech](https://img.shields.io/badge/Stack-Jenkins%20|%20K8s%20|%20Docker%20|%20Gemini%20AI-blue)

## 🌟 Overview
This project demonstrates a professional-grade transition from manual L1 Support operations to a fully automated, Cloud-Native CI/CD ecosystem. It features an **"AI Log Whisperer"**—an agentic integration using the Gemini 1.5 Pro API that automatically diagnoses pipeline failures in real-time.

## 🏗️ Architecture
The pipeline follows a modern "Push-to-Deploy" workflow:
1. **Developer Push:** Code is pushed to GitHub.
2. **Instant Trigger:** GitHub Webhook (via ngrok tunnel) notifies Jenkins.
3. **Containerization:** Jenkins builds a Docker image and pushes it to Docker Hub.
4. **Orchestration:** Jenkins triggers a rolling update on a **KIND (Kubernetes-in-Docker)** cluster.
5. **AI Supervision:** If any stage fails, a Post-Build action triggers a **Gemini AI Agent** to perform Root Cause Analysis (RCA) on the Jenkins console logs.

## 🛠️ Tech Stack
- **CI/CD:** Jenkins (Pipeline-as-Code)
- **Containerization:** Docker
- **Orchestration:** Kubernetes (KIND)
- **AI Engine:** Google Gemini API (Agentic RCA)
- **Networking:** ngrok (Webhook Tunneling)
- **Environment:** WSL2 (Ubuntu)

## 🚀 Key Features
### 1. Agentic AI Error Handling
Unlike standard pipelines that just show a "Red" status, this pipeline explains *why* it failed. Using a custom Bash/JQ integration, the pipeline sends failed logs to Gemini to receive a 3-step remediation plan directly in the Jenkins console.

### 2. High Availability
The Kubernetes manifest (`deployment.yaml`) ensures 2 replicas are always running. It utilizes an `Always` image pull policy and `rollout restart` logic to ensure zero-downtime updates.

### 3. Automated Webhooks
Integrated real-time triggers using GitHub Webhooks, moving away from inefficient SCM polling to event-driven architecture.

## 📂 Project Structure
\`\`\`text
.
├── app/
│   └── index.html         # Frontend Application
├── docker/
│   └── Dockerfile         # Container Blueprint
├── k8s/
│   └── deployment.yaml    # Kubernetes Manifests
└── Jenkinsfile            # Pipeline-as-Code Definition
\`\`\`

---
**Developed by Malay Panigrahi** | *L1 Support Engineer @ Axcend Automation*
