pipeline {
    agent none 

    stages {
        // --- STAGE 1: CI (The Cooking Phase) ---
        stage('Build & Test') {
            agent {
                docker { 
                    image 'node:20-alpine' 
                    args '-v /tmp:/tmp -e HOME=${WORKSPACE}'
                }
            }
            steps {
                echo 'CI: Installing and Testing inside Docker...'
                sh 'npm ci'
                sh 'npm test -- --watchAll=false'
                sh 'npm run build'
                
                // Saving the artifacts to move them to the WSL host
                stash includes: 'build/**', name: 'app-artifacts'
            }
        }

        // --- STAGE 2: DELIVERY GATE (The Human Check) ---
        stage('Quality Gate') {
            steps {
                echo 'Waiting for IT Officer approval...'
                timeout(time: 1, unit: 'HOURS') {
                    input message: 'Do you want to deploy this build to Production?', 
                          ok: 'Approve & Deploy'
                }
            }
        }

        // --- STAGE 3: CD (The Serving Phase) ---
        stage('Deploy') {
            agent any // Runs on your WSL host machine
            steps {
                echo 'CD: Deploying to WSL Host...'
                
                // 1. Clean up old files and Kill any old process on Port 3000
                // 'fuser' is a standard Linux tool to manage port processes
                sh 'rm -rf ./build'
                sh 'fuser -k 3000/tcp || true'
                
                // 2. Retrieve the compiled files
                unstash 'app-artifacts'
                
                // 3. Start the application using the full path
                // We redirect output to 'deploy.log' so you can debug if it fails
                sh 'nohup /usr/bin/npx serve -s build -l 3000 > deploy.log 2>&1 &'
                
                // 4. Health Check: Wait for the app to start and verify it's up
                echo 'Running Health Check...'
                sleep 5 // Give the server 5 seconds to wake up
                sh 'curl -s --retry 3 http://localhost:3000 > /dev/null'
                
                echo 'SUCCESS: Application is live at http://localhost:3000'
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed! Check deploy.log or Jenkins Console Output.'
        }
        aborted {
            echo 'Deployment was cancelled.'
        }
    }
}
