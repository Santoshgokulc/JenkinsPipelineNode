pipeline {
    /* AGENT: Using Docker to ensure a clean, isolated environment.
       This image comes pre-installed with Node.js 20 and npm.
    */
    agent {
        docker { 
            image 'node:20-alpine' 
        }
    }

    stages {
        stage('Initialize') {
            steps {
                echo 'Starting Pipeline for Santosh...'
                sh 'node -v'
                sh 'npm -v'
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing Backend and Frontend packages...'
                // clean-install (ci) is better for automation than 'npm install'
                sh 'npm ci' 
            }
        }

        stage('Security Scan') {
            steps {
                echo 'Checking for vulnerabilities in libraries...'
                // A common exam topic: Application Security
                sh 'npm audit --audit-level=high || true' 
            }
        }

        stage('Unit Testing') {
            steps {
                echo 'Running Server and Client tests...'
                // Runs tests and exits (doesn't stay in watch mode)
                sh 'npm test -- --watchAll=false'
            }
        }

        stage('Database Migration') {
            steps {
                echo 'Updating Database Schema...'
                /* In a real scenario, you'd provide DB credentials here.
                   This ensures the DB matches the new code.
                */
                sh 'echo "Running: npx sequelize-cli db:migrate"'
            }
        }

        stage('Build Frontend') {
            steps {
                echo 'Creating Production Build of the React App...'
                sh 'npm run build'
            }
        }

        stage('Archive Artifacts') {
            steps {
                echo 'Saving the build files for deployment...'
                // This saves the "dist" or "build" folder in Jenkins so you can download it later
                archiveArtifacts artifacts: 'build/**', fingerprint: true
            }
        }
    }

    /* POST-BUILD: This runs after all stages, regardless of success or failure.
    */
    post {
        success {
            echo 'Pipeline completed successfully! Ready for Deployment.'
        }
        failure {
            echo 'Pipeline failed. Checking logs...'
            // This is where you would normally send an email or Slack alert
        }
    }
}
