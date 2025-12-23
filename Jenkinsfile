node {
    def mavenHome
    def mavenCMD
    def dockerHome
    def dockerCMD
    def tagName = "3.0"
    
    stage('Prepare Environment') {
        echo 'checking tools...'
        mavenHome = tool name: 'maven-1', type: 'maven'
        mavenCMD = "${mavenHome}/bin/mvn"
        
        // Ensure Docker is configured in Global Tool Configuration
        dockerCMD = "docker"
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
        // CHANGE: Use usernamePassword instead of string
        withCredentials([usernamePassword(credentialsId: 'dock-password', 
                                          usernameVariable: 'DOCKER_USER', 
                                          passwordVariable: 'DOCKER_PASS')]) {
            
            // Secure login using the injected variables
            sh "echo ${DOCKER_PASS} | ${dockerCMD} login -u ${DOCKER_USER} --password-stdin"
            
            // Push the image using your specific tag
            sh "${dockerCMD} push sdfa777/insurance-star_agile-project-3:${tagName}"
        }
    }

    stage('Approval for EKS') {
        // Pauses execution until a user clicks 'Approve' in Jenkins UI
        input message: "Deploy version ${tagName} to EKS Cluster?", ok: "Deploy Now"
    }

    stage('Deploy to EKS') {
        echo 'Deploying to Kubernetes...'
        sh "aws eks update-kubeconfig --region ap-south-1 --name capstone-project"
        
        // This command applies EVERY file inside the kubernetes folder
        sh "kubectl apply -f kubernetes/"
        
        echo 'Deployment complete!'
    }
        
        echo 'Deployment complete! Check status with: kubectl get pods'
    }
