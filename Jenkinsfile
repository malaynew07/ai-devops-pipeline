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
                // 1. Capture the last 50 lines of logs using Groovy
                def logContent = currentBuild.rawBuild.getLog(50).join('\n')
                writeFile file: 'error_log.txt', text: logContent

                // 2. The AI Logic - Using Shell for API interaction
                sh """
                #!/bin/bash
                # Clean logs: remove double quotes and backslashes so JSON stays valid
                CLEAN_LOGS=\$(cat error_log.txt | tr -d '"' | tr -d '\\\\')

                # Create the JSON payload file
                cat <<EOF > payload.json
{
  "contents": [
    {
      "parts": [
        {
          "text": "Act as a Senior DevOps Engineer. The Jenkins pipeline failed. Analyze these logs and give a 3-step fix: \$CLEAN_LOGS"
        }
      ]
    }
  ]
}
EOF

                # 3. Call the API using the exact model from your list (Gemini 3.1)
                RESPONSE=\$(curl -s -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-3.1-flash-lite-preview:generateContent?key=${GEMINI_API_KEY}" \\
                    -H 'Content-Type: application/json' \\
                    -d @payload.json)

                echo -e "\\n================ AI DIAGNOSIS & REMEDIATION ================\\n"

                # Check if the response contains an error
                if echo "\$RESPONSE" | jq -e '.error' > /dev/null; then
                    echo "AI Agent Error Details:"
                    echo "\$RESPONSE" | jq .
                else
                    # Successfully extract the text
                    echo "\$RESPONSE" | jq -r '.candidates[0].content.parts[0].text'
                fi

                echo -e "\\n============================================================\\n"
                """
            }
        }
        success {
            echo "🎉 Pipeline completed successfully! The AI Agent is resting."
        }
    }
}
