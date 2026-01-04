node {
    def mavenHome
    def mavenCMD
    def dockerCMD = "docker"
    def tagName = "3.0"
    
    stage('Prepare Environment') {
        echo 'Initializing tools...'
        mavenHome = tool name: 'maven-1', type: 'maven'
        mavenCMD = "${mavenHome}/bin/mvn"
    }
    
    stage('Git Checkout') {
        git 'https://github.com/karthick-nagarajulu/star-agile-insurance-project.git'
    }

    stage('Build & Package') {
        sh "${mavenCMD} clean package -DskipTests" 
    }
    
    stage('Docker Build') {
        sh "${dockerCMD} build -t sdfa777/insurance-star_agile-project-3:${tagName} ."
    }
    
    stage('Push to DockerHub') {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials-id', 
                                          usernameVariable: 'DOCKER_USER', 
                                          passwordVariable: 'DOCKER_PASS')]) {
            sh "echo ${DOCKER_PASS} | ${dockerCMD} login -u ${DOCKER_USER} --password-stdin"
            sh "${dockerCMD} push sdfa777/insurance-star_agile-project-3:${tagName}"
        }
    }

    stage('Deploy to Worker') {
            steps {
                // Ensure 'medicure-ssh-key' exists in Jenkins Credentials as 'SSH Username with private key'
                sshagent(credentials: ['key']) {
                    sh """
ssh -o StrictHostKeyChecking=no ubuntu@${env.jenkins} << 'EOF'
    docker pull ${DOCKER_LATEST}
    docker stop health-app || true
    docker rm health-app || true
    docker run -d --name health-app -p 8081:8080 ${DOCKER_LATEST}
EOF
                    """
                }
            }
        }

   stage('Deploy to Kubernetes') {
    echo 'Deploying to Self-Managed Cluster...'
    withCredentials([file(credentialsId: 'k8s-config', variable: 'KUBECONFIG_FILE')]) {
        // This command confirmed the name is 'insurance-app'
        sh 'kubectl --kubeconfig=$KUBECONFIG_FILE apply -f kubernetes/'
        
        // Use the correct name: insurance-app
        // Also added the correct container name usually found in these projects: insurance-container
        sh "kubectl --kubeconfig=\$KUBECONFIG_FILE set image deployment/insurance-app insurance-container=sdfa777/insurance-star_agile-project-3:${tagName}"
        
        echo 'Deployment complete!'
    }
}
}
