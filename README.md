# DevSecOps Automated Pipeline

A complete Secure CI/CD pipeline built on AWS EC2 that integrates security at every stage. This project demonstrates how to automate the build, scan, and deployment of a Node.js application using modern DevSecOps tools.

## Pipeline Features
- **CI/CD Engine:** Jenkins (Self-hosted on Ubuntu)
- **Security Scanning (SCA):** Snyk (Scans dependencies for vulnerabilities)
- **Containerization:** Docker (Multi-stage build)
- **Container Scanning:** Trivy (Scans Docker images for OS-level vulnerabilities)
- **Secret Management:** HashiCorp Vault (Injects runtime secrets into containers)
- **Application:** Node.js + Express.js

##  Architecture
1. **Developer** pushes code to GitHub.
2. **Jenkins** triggers the pipeline automatically.
3. **Snyk** checks `package.json` for vulnerable libraries.
4. **Docker** builds the application image.
5. **Trivy** scans the image; the build fails if "CRITICAL" issues are found.
6. **Vault** provides the Database Password at runtime.
7. **Deployment** starts the container on port 3000.

##  Prerequisites
- AWS EC2 Instance (Ubuntu 22.04/24.04)
- Docker, Jenkins, Snyk CLI, and Trivy installed
- HashiCorp Vault running in `-dev` mode

##  Setup Instructions

 Start the Vault Server
```bash
vault server -dev -dev-root-token-id="root" &
export VAULT_ADDR='http://127.0.0.1:8200'
vault kv put secret/devsecops-app password=" "
```

### 2. Configure Jenkins Permissions
Ensure Jenkins can run Docker commands:
```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

### 3. Jenkins Pipeline Configuration
Create a Pipeline job in Jenkins.
Link your GitHub repository.
Use the provided Jenkinsfile in the repository root.

## Security Quality Gates
This pipeline is configured to:
Snyk: Identify and report vulnerabilities in npm packages.
Trivy: Block the deployment if the Docker image contains CRITICAL vulnerabilities.
Vault: Ensure no sensitive passwords or API keys are hardcoded in the source code.



##  AUTHOR: PHILIP LUCKY
EMAIL: philipslucky24@gmail.com
