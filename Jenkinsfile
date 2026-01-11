pipeline {
    agent none // We define specific agents for each stage

    stages {
        // --- STAGE 1: CI (The Cooking Phase) ---
        stage('Build & Test') {
            agent {
                docker { 
                    image 'node:20-alpine' 
                    // Fixes the permission/cache issue we encountered earlier
                    args '-v /tmp:/tmp -e HOME=${WORKSPACE}'
                }
            }
            steps {
                echo 'CI: Installing and Testing inside Docker...'
                sh 'npm ci'
                sh 'npm test -- --watchAll=false'
                sh 'npm run build'
                
                // Saving the "Lunchbox" to move it out of the Docker container
                stash includes: 'build/**', name: 'app-artifacts'
            }
        }

        // --- STAGE 2: DELIVERY GATE (The Human Check) ---
        stage('Quality Gate') {
            steps {
                echo 'Waiting for IT Officer approval...'
                // This creates the Proceed (OK) and Abort (Cancel) buttons
                // It will automatically cancel if no one clicks after 1 hour
                timeout(time: 1, unit: 'HOURS') {
                    input message: 'Do you want to deploy this build to Production?', 
                          ok: 'Approve & Deploy'
                }
            }
        }

        // --- STAGE 3: CD (The Serving Phase) ---
        stage('Deploy') {
            agent any // This runs directly on your WSL host machine
            steps {
                echo 'CD: Deploying artifacts to the Web Server...'
                
                // 1. Clean up old files on the host
                sh 'rm -rf ./deploy_folder'
                
                // 2. Retrieve the "Lunchbox" we saved in Stage 1
                unstash 'app-artifacts'
                
                // 3. Start the application in the background
                // Using 'serve' as an example; for your exam, think of this as 'starting the service'
                sh 'nohup npx serve -s build -l 3000 & > /dev/null'
                
                echo 'SUCCESS: Application is live at http://localhost:3000'
            }
        }
    }

    // --- POST-BUILD NOTIFICATIONS ---
    post {
        always {
            echo 'Cleaning up workspace...'
        }
        failure {
            echo 'Pipeline failed! Please check the console output for errors.'
        }
        aborted {
            echo 'Deployment was cancelled by the user.'
        }
    }
}
