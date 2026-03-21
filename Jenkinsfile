pipeline {
    agent any

    environment {
        // We will configure these credentials in Jenkins next
        DOCKERHUB_CREDS = credentials('dockerhub-credentials')
        GEMINI_API_KEY = credentials('gemini-api-key')
        
        // Change this to your actual Docker Hub username!
        IMAGE_NAME = "malaynew07/ai-devops-app"
        IMAGE_TAG = "v${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
                echo "✅ Code pulled successfully from Git."
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -t ${IMAGE_NAME}:latest -f docker/Dockerfile ."
                echo "✅ Docker image built."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                // Log in securely and push both the versioned tag and the 'latest' tag
                sh "echo \$DOCKERHUB_CREDS_PSW | docker login -u \$DOCKERHUB_CREDS_USR --password-stdin"
                sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                sh "docker push ${IMAGE_NAME}:latest"
                echo "✅ Image pushed to Docker Hub."
            }
        }

        stage('Deploy to KIND Cluster') {
            steps {
                // Dynamically update the deployment.yaml with the exact new image tag
                sh "sed -i 's|ai-devops-app:latest|${IMAGE_NAME}:${IMAGE_TAG}|g' k8s/deployment.yaml"
                
                // Apply the updated manifest to the agentic-devops namespace
                sh "kubectl apply -f k8s/deployment.yaml"
                echo "✅ Deployment triggered on local KIND cluster."
            }
        }
    }

    // ==========================================
    // THE AGENTIC AI INTEGRATION
    // ==========================================
    post {
        failure {
            echo "❌ Pipeline failed! Waking up the AI DevOps Agent for Root Cause Analysis..."
            script {
                // 1. Grab the last 50 lines of the failed Jenkins build log
                def logContent = currentBuild.rawBuild.getLog(50).join('\n')
                writeFile file: 'error_log.txt', text: logContent
                
                // 2. Execute our Agentic AI Bash Script
                sh """
                #!/bin/bash
                echo "🤖 Agent: Analyzing logs via Gemini API..."
                
                PROMPT="Act as a Senior DevOps Engineer. The Jenkins pipeline just failed. Analyze these logs, identify the root cause, and give a concise 3-step action plan to resolve it. Logs: \$(cat error_log.txt)"
                
                # Format payload for the API
                JSON_PAYLOAD=\$(jq -n --arg text "\$PROMPT" '{contents: [{parts: [{"text": \$text}]}]}')
                
                # Call the AI Model
                RESPONSE=\$(curl -s -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-pro:generateContent?key=${GEMINI_API_KEY}" \
                    -H 'Content-Type: application/json' \
                    -d "\$JSON_PAYLOAD")
                    
                # Print the AI's diagnosis cleanly to the Jenkins console
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
