pipeline {
    agent any

    environment {
        DOCKERHUB_CREDS = credentials('dockerhub-credentials')
        GEMINI_API_KEY = credentials('gemini-api-key')
        IMAGE_NAME = "malaynew07/ai-devops-app"
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
                echo "✅ Code pulled successfully from Git."
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                sh "echo \$DOCKERHUB_CREDS_PSW | docker login -u \$DOCKERHUB_CREDS_USR --password-stdin"
                sh "docker build -t ${IMAGE_NAME}:latest -f docker/Dockerfile ."
                sh "docker push ${IMAGE_NAME}:latest"
                echo "✅ New image pushed to Docker Hub."
            }
        }

        stage('Deploy to KIND Cluster') {
            steps {
                sh "kubectl apply -f k8s/deployment.yaml"
                sh "kubectl rollout restart deployment/web-app-deployment -n agentic-devops"
                echo "✅ Deployment updated in KIND cluster."
            }
        }
    }

    post {
        failure {
            echo "❌ Pipeline failed! Waking up the AI DevOps Agent..."
            script {
                // 1. Grab logs using Groovy
                def logContent = currentBuild.rawBuild.getLog(50).join('\n')
                writeFile file: 'error_log.txt', text: logContent

                // 2. Run AI Logic - Escaping dollar signs for Bash variables
                sh """
                #!/bin/bash
                # Escape the $ signs so Groovy doesn't try to parse them
                PROMPT="Act as a Senior DevOps Engineer. The Jenkins pipeline just failed. Analyze these logs, identify the root cause, and give a 3-step fix. Logs: \$(cat error_log.txt)"
                
                PAYLOAD=\$(jq -n --arg text "\$PROMPT" '{contents: [{parts: [{"text": \$text}]}]}')
                
                RESPONSE=\$(curl -s -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash-latest:generateContent?key=${GEMINI_API_KEY}" \
                    -H 'Content-Type: application/json' \
                    -d "\$PAYLOAD")
                
                echo -e "\\n================ AI DIAGNOSIS & REMEDIATION ================\\n"
                echo "\$RESPONSE" | jq -r '.candidates[0].content.parts[0].text'
                echo -e "\\n============================================================\\n"
                """
            }
        }
        success {
            echo "🎉 Pipeline completed successfully! The AI Agent is resting."
        }
    }
}
