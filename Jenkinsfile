pipeline {
    agent any

    environment {
        // Points to your Vault instance
        VAULT_ADDR = "http://127.0.0.1:8200"
        VAULT_TOKEN = "root"
        IMAGE_NAME = "devsecops-app"
    }

    stages {
        stage('Clone Code') {
            steps {
                // Replace with your actual repository URL
                git branch: 'main', url: 'https://github.com/Philip-Lucky/DevSecOps-project'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Snyk Security Scan') {
            steps {
                // Using '|| true' allows the pipeline to continue if vulnerabilities are found
                // Remove '|| true' if you want the build to FAIL on security issues
                sh 'snyk test --severity-threshold=high || true'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }

        stage('Trivy Container Scan') {
            steps {
                // Scans the image we just built
                sh "trivy image --exit-code 0 --severity HIGH,CRITICAL ${IMAGE_NAME}"
            }
        }

        stage('Fetch Secrets & Deploy') {
            steps {
                sh '''
                # 1. Fetch the secret from Vault
                DB_PASSWORD=$(vault kv get -field=password secret/devsecops-app)

                # 2. Stop and remove the old container if it exists (prevents port conflicts)
                docker rm -f ${IMAGE_NAME} || true

                # 3. Run the new container, passing the secret as an environment variable
                docker run -d \
                  --name ${IMAGE_NAME} \
                  -p 3000:3000 \
                  -e APP_DB_PASSWORD=$DB_PASSWORD \
                  ${IMAGE_NAME}
                '''
            }
        }
    }

    post {
        always {
            echo "Cleaning up workspace..."
            cleanWs()
        }
        success {
            echo "Deployment successful! Application is running on port 3000."
        }
        failure {
            echo "Pipeline failed. Check the console output for errors."
        }
    }
}

