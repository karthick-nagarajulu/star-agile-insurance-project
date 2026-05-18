pipeline {
    agent any
    triggers {
        githubPush()
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/karthick-nagarajulu/star-agile-insurance-project.git'
            }
        }
        stage('Build') {
            steps { echo 'Building...' }
        }
    }
}
