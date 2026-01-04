node {
    def mavenHome
    def mavenCMD
    def dockerCMD = "docker"
    def tagName = "3.0"
    
    stage('Prepare Environment') {
        mavenHome = tool name: 'maven-1', type: 'maven'
        mavenCMD = "${mavenHome}/bin/mvn"
    }
    
    stage('Git Checkout') {
        git 'https://github.com/karthick-nagarajulu/star-agile-insurance-project.git'
    }

    stage('Build & Package') {
        sh "${mavenCMD} clean package -DskipTests" 
    }
    
    stage('Docker Build & Push') {
        sh "${dockerCMD} build -t sdfa777/insurance-star_agile-project-3:${tagName} ."
        withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials-id', usernameVariable: 'U', passwordVariable: 'P')]) {
            sh "echo ${P} | ${dockerCMD} login -u ${U} --password-stdin"
            sh "${dockerCMD} push sdfa777/insurance-star_agile-project-3:${tagName}"
        }
    }

    stage('Deploy to Kubernetes') {
        echo 'Deploying to Self-Managed Cluster...'
        withCredentials([file(credentialsId: 'k8s-config', variable: 'KUBECONFIG_FILE')]) {
            // Apply all kubernetes manifests
            sh "kubectl --kubeconfig=${KUBECONFIG_FILE} apply -f kubernetes/"
            
            // Fixed: Changed insurance-deployment to insurance-app based on previous success logs
            sh "kubectl --kubeconfig=${KUBECONFIG_FILE} set image deployment/insurance-app insurance-container=sdfa777/insurance-star_agile-project-3:${tagName}"
            
            echo 'Deployment complete!'
        }
    }
}
