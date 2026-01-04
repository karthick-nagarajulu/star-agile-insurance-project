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

    stage('Unit Tests') {
        sh "${mavenCMD} test"
    }
    
    stage('Build & Package') {
        sh "${mavenCMD} clean package -DskipTests" 
    }
    
    stage('Docker Build') {
        // Using your DockerHub username 'sdfa777'
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

    stage('Approval') {
        input message: "Deploy version ${tagName} to Cluster?", ok: "Deploy Now"
    }

    stage('Deploy to Kubernetes') {
        echo 'Deploying to Self-Managed Cluster...'
        // This block securely provides the kubeconfig to the kubectl command
        withCredentials([file(credentialsId: 'k8s-config', variable: 'KUBECONFIG_FILE')]) {
            // Apply all manifests in your 'kubernetes/' folder
            sh "kubectl --kubeconfig=${KUBECONFIG_FILE} apply -f kubernetes/"
            
            // Force the deployment to use the new image version
            sh "kubectl --kubeconfig=${KUBECONFIG_FILE} set image deployment/insurance-deployment insurance-container=sdfa777/insurance-star_agile-project-3:${tagName} --record"
            
            echo 'Deployment complete!'
        }
    }
}
