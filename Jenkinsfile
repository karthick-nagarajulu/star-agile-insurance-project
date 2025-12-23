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
        
    
        dockerCMD = "docker"
    }
    
    stage('Git Checkout') {
        git 'https://github.com/karthick-nagarajulu/star-agile-insurance-project.git'
    }

    stage('Unit Tests') {
        echo 'Running Maven Tests...'
        sh "${mavenCMD} test"
    }
    
    stage('Build & Package') {
        echo 'Compiling and Packaging Application...'
        sh "${mavenCMD} clean package -DskipTests" 
    }
    
    stage('Publish Reports') {
        // Only works if "HTML Publisher" plugin is installed
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
        withCredentials([usernamePassword(credentialsId: 'dock-password', 
                                          usernameVariable: 'DOCKER_USER', 
                                          passwordVariable: 'DOCKER_PASS')]) {
            sh "echo ${DOCKER_PASS} | ${dockerCMD} login -u ${DOCKER_USER} --password-stdin"
            sh "${dockerCMD} push sdfa777/insurance-star_agile-project-3:${tagName}"
        }
    }

    stage('Approval for EKS') {
        input message: "Deploy version ${tagName} to EKS Cluster?", ok: "Deploy Now"
    }

   stage('Deploy to EKS') {
        echo 'Deploying to Kubernetes...'
        // Clean workspace config to force a fresh permission check
        sh "rm -f ${WORKSPACE}/.kubeconfig"
        withEnv(["KUBECONFIG=${WORKSPACE}/.kubeconfig"]) {
            sh "aws eks update-kubeconfig --region ap-south-1 --name capstone-project --kubeconfig ${WORKSPACE}/.kubeconfig"
            
            // Apply the files
            sh "kubectl apply -f kubernetes/ --kubeconfig ${WORKSPACE}/.kubeconfig"
            
            echo 'Deployment complete!'
        }
    }
}
