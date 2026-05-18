pipeline {
    agent any
    triggers {
        githubPush()
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/karthick-nagarajulu/star-agile-insurance-project.git'
            }
        }
        stage('Build') {
            steps { echo 'Building...' }
        }
    }
}
