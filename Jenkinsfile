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
        withCredentials([usernamePassword(credentialsId: 'dock-password', 
                                          usernameVariable: 'DOCKER_USER', 
                                          passwordVariable: 'DOCKER_PASS')]) {
            sh "echo ${DOCKER_PASS} | ${dockerCMD} login -u ${DOCKER_USER} --password-stdin"
            sh "${dockerCMD} push sdfa777/insurance-star_agile-project-3:${tagName}"
        }
    }

    stage('Deploy to Kubernetes') {
        echo 'Deploying to Self-Managed Cluster...'
        // This binds the 'k8s-config' secret file to a temporary variable $KUBECONFIG
        withCredentials([file(credentialsId: 'k8s-config', variable: 'KUBECONFIG')]) {
            // Apply the YAML files from your github folder
            sh "kubectl --kubeconfig=${KUBECONFIG} apply -f kubernetes/"
            
            // Specifically update the image to the new tag
            sh "kubectl --kubeconfig=${KUBECONFIG} set image deployment/insurance-deployment insurance-container=sdfa777/insurance-star_agile-project-3:${tagName}"
            
            echo 'Deployment complete!'
        }
    }
}
