pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                echo "Building from branch ${env.BRANCH_NAME ?: 'default'}"
                echo "Workspace: ${env.WORKSPACE}"
                sh 'ls -la'              // show files Jenkins checked out
            }
        }
        
        stage('Test') {
            steps {
                echo 'Running tests...'
                // sh 'mvn test'    // uncomment when you have a real project
            }
        }
        
        stage('Done') {
            steps {
                echo "Build #${env.BUILD_NUMBER} completed ✅"
            }
        }
    }
    
    post {
        success { echo '✅ Pipeline succeeded' }
        failure { echo '❌ Pipeline failed' }
    }
}
