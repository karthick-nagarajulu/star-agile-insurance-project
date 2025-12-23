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
        // We set KUBECONFIG specifically for the jenkins user's workspace
        withEnv(["KUBECONFIG=${WORKSPACE}/.kube/config"]) {
            // 1. Create the directory for the config
            sh "mkdir -p ${WORKSPACE}/.kube"
            
            // 2. Generate a fresh kubeconfig file that Jenkins definitely has permission to read
            sh "aws eks update-kubeconfig --region ap-south-1 --name capstone-project --kubeconfig ${WORKSPACE}/.kube/config"
            
            // 3. Apply the deployment using the local config
            sh "kubectl apply -f kubernetes/ --kubeconfig ${WORKSPACE}/.kube/config --validate=false"
            
            echo 'Deployment complete!'
        }
    }
