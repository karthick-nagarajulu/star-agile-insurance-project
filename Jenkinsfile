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

    stage('Deploy to Kubernetes') {
    echo 'Deploying to Self-Managed Cluster...'
    withCredentials([file(credentialsId: 'k8s-config', variable: 'KUBECONFIG_FILE')]) {
        // 1. Use single quotes ('') instead of double quotes ("") to fix the security warning
        // 2. Ensure the deployment name (insurance-deployment) matches 'kubectl get deployments'
        sh 'kubectl --kubeconfig=$KUBECONFIG_FILE apply -f kubernetes/'
        sh 'kubectl --kubeconfig=$KUBECONFIG_FILE set image deployment/insurance-deployment insurance-container=sdfa777/insurance-star_agile-project-3:${tagName}'
        
        echo 'Deployment complete!'
    }
}
}
