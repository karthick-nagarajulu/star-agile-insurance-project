node {
    def mavenHome
    def mavenCMD
    def dockerHome
    def dockerCMD
    def tagName = "3.0"
    
    stage('Prepare Environment') {
        echo 'Initializing tools...'
        mavenHome = tool name: 'maven-1', type: 'maven'
        mavenCMD = "${mavenHome}/bin/mvn"
        
        // Ensure Docker is configured in Global Tool Configuration
        dockerHome = tool name: 'docker', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
        dockerCMD = "${dockerHome}/bin/docker"
    }
    
    stage('Git Checkout') {
        git 'https://github.com/karthick-nagarajulu/star-agile-insurance-project.git'
    }

    stage('Unit Tests') {
        echo 'Running Maven Tests...'
        // This generates the surefire reports used in the next stage
        sh "${mavenCMD} test"
    }
    
    stage('Build & Package') {
        echo 'Compiling and Packaging Application...'
        sh "${mavenCMD} clean package -DskipTests" // Tests already passed
    }
    
    stage('Publish Reports') {
        publishHTML([
            allowMissing: false, 
            alwaysLinkToLastBuild: false, 
            keepAll: true, 
            reportDir: 'target/surefire-reports', 
            reportFiles: 'index.html', 
            reportName: 'JUnit Test Report'
        ])
    }
    
    stage('Docker Build') {
        sh "${dockerCMD} build -t sdfa777/insurance-star_agile-project-3:${tagName} ."
    }
    
    stage('Push to DockerHub') {
        withCredentials([string(credentialsId: 'dock-password', variable: 'dockerHubPassword')]) {
            sh "${dockerCMD} login -u sdfa777 -p ${dockerHubPassword}"
            sh "${dockerCMD} push sdfa777/insurance-star_agile-project-3:${tagName}"
        }
    }

    stage('Approval for EKS') {
        // Pauses execution until a user clicks 'Approve' in Jenkins UI
        input message: "Deploy version ${tagName} to EKS Cluster?", ok: "Deploy Now"
    }

    stage('Deploy to EKS') {
        echo 'Deploying to Kubernetes...'
        // 1. Update kubeconfig to point to your cluster
        sh "aws eks update-kubeconfig --region ap-south-1 --name capstone-project"
        
        // 2. Apply your K8s deployment and service files
        // Ensure deployment.yaml uses the ${tagName} variable or 'latest'
        sh "kubectl apply -f kubernetes/deployment.yaml"
        sh "kubectl apply -f kubernetes/service.yaml"
        
        echo 'Deployment complete! Check status with: kubectl get pods'
    }
}
