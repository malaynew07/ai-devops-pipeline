pipeline {
    agent any

    environment {
        DOCKERHUB_CREDS = credentials('dockerhub-credentials')
        GEMINI_API_KEY = credentials('gemini-api-key')
        IMAGE_NAME = "malaynew07/ai-devops-app"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Push Image') {
            steps {
                sh "echo \$DOCKERHUB_CREDS_PSW | docker login -u \$DOCKERHUB_CREDS_USR --password-stdin"
                sh "docker build -t ${IMAGE_NAME}:latest -f docker/Dockerfile ."
                sh "docker push ${IMAGE_NAME}:latest"
            }
        }

        stage('Deploy to K8s') {
            steps {
                sh "kubectl apply -f k8s/deployment.yaml"
                sh "kubectl rollout restart deployment/web-app-deployment -n agentic-devops"
            }
        }
    }

    post {
        failure {
            script {
                // 1. Get the last 50 lines of the error log
                def logs = currentBuild.rawBuild.getLog(50).join('\n').replace('"', '').replace('\\', '')
                
                echo "❌ Build Failed. Asking AI for the solution..."

                // 2. Simple, Direct AI Analysis
                sh """
                curl -s -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-3.1-flash-lite-preview:generateContent?key=${GEMINI_API_KEY}" \
                    -H 'Content-Type: application/json' \
                    -d '{
                      "contents": [{
                        "parts":[{"text": "Act as a DevOps Expert. Analyze these logs and provide a clear 3-step fix only: ${logs}"}]
                      }]
                    }' > ai_response.json

                echo -e "\\n--- AI FIX STEPS ---"
                # This Python line safely prints the AI's solution
                python3 -c "import json, sys; print(json.load(open('ai_response.json'))['candidates'][0]['content']['parts'][0]['text'])"
                echo -e "----------------------\\n"
                """
            }
        }
        success {
            echo "✅ Success! Application is live."
        }
    }
}
