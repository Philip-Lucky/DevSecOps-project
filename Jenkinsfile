pipeline {
    agent any

    environment {
        VAULT_ADDR = "http://127.0.0.1:8200"
        VAULT_TOKEN = "root"
        IMAGE_NAME = "devsecops-app"
        // It is safer to put the token in a variable
        SNYK_TOKEN = "snyk_uat.1fcad39e.eyJlIjoxNzc0OTQ3NzEwLCJoIjoic255ay5pbyIsImoiOiJBWjBmRktzdEFHWUE2MmNjN0hqeWlnIiwicyI6ImU3MUlXYmR6VGdhdkF3clNUWENBQlEiLCJ0aWQiOiJBQUFBQUFBQUFBQUFBQUFBQUFBQUFBIn0.q3lKoNxSJFJgAALsAgNaVQ6Qut-iJt7-UxcZlFzrBmR40DE11B-dRI86QJr4eZLYQX5NnI8cSGXf4R9YbBcQCQ"
    }

    stages {
        // I removed the "Clone Code" stage because Jenkins does this automatically
        // when you use "Pipeline script from SCM"

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Snyk Security Scan') {
            steps {
                // Using the variable we defined in environment
                sh "SNYK_TOKEN=${SNYK_TOKEN} snyk test --severity-threshold=high || true"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }

        stage('Trivy Container Scan') {
            steps {
                sh "trivy image --exit-code 0 --severity HIGH,CRITICAL ${IMAGE_NAME}"
            }
        }

        stage('Fetch Secrets & Deploy') {
            steps {
                sh '''
                # 1. Fetch the secret from Vault
                DB_PASSWORD=$(vault kv get -field=password secret/data/devsecops-app)
                
                # 2. Stop and remove old container if it exists
                docker rm -f devsecops-app || true
                
                # 3. Run the new container with the secret
                docker run -d --name devsecops-app -p 3000:3000 -e APP_PASSWORD=$DB_PASSWORD devsecops-app
                '''
            }
        }
    }
}

