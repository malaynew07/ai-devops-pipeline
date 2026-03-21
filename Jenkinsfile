pipeline {
    agent any

    environment {
        DOCKERHUB_CREDS = credentials('dockerhub-credentials')
        GEMINI_API_KEY = credentials('gemini-api-key')
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
                sh "echo \$DOCKERHUB_CREDS_PSW | docker login -u \$DOCKERHUB_CREDS_USR --password-stdin"
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -t ${IMAGE_NAME}:latest -f docker/Dockerfile ."
                echo "✅ Docker image built."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                sh "docker push ${IMAGE_NAME}:latest"
                echo "✅ Image pushed to Docker Hub."
            }
        }

        stage('Deploy to KIND Cluster') {
            steps {
                script {
                    sh "sed -i 's|image: .*ai-devops-app:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|g' k8s/deployment.yaml"
                }
                sh "cat k8s/deployment.yaml"
                sh "kubectl apply -f k8s/deployment.yaml"
                echo "✅ Deployment triggered on local KIND cluster."
            }
        }
    }

    post {
        failure {
            echo "❌ Pipeline failed! Waking up the AI DevOps Agent for Root Cause Analysis..."
            script {
                def logContent = currentBuild.rawBuild.getLog(50).join('\n')
                writeFile file: 'error_log.txt', text: logContent

                sh """
                #!/bin/bash
                echo "🤖 Agent: Analyzing logs via Gemini API..."
                PROMPT="Act as a Senior DevOps Engineer. The Jenkins pipeline just failed. Analyze these logs, identify the root cause, and give a concise 3-step action plan to resolve it. Logs: \$(cat error_log.txt)"
                JSON_PAYLOAD=\$(jq -n --arg text "\$PROMPT" '{contents: [{parts: [{"text": \$text}]}]}')
                RESPONSE=\$(curl -s -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-pro:generateContent?key=${GEMINI_API_KEY}" \
                    -H 'Content-Type: application/json' \
                    -d "\$JSON_PAYLOAD")
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
} // <-- This is the one that was missing to close the pipeline!
