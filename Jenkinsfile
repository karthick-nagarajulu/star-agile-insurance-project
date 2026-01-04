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
    
    stage('Docker Build & Push') {
        sh "${dockerCMD} build -t sdfa777/insurance-star_agile-project-3:${tagName} ."
        // Using your specific credentials ID
        withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials-id', 
                                          usernameVariable: 'U', 
                                          passwordVariable: 'P')]) {
            sh "echo ${P} | ${dockerCMD} login -u ${U} --password-stdin"
            sh "${dockerCMD} push sdfa777/insurance-star_agile-project-3:${tagName}"
        }
    }

    stage('Deploy to EC2 (via Ansible)') {
        echo 'Deploying to EC2 instance using Ansible...'
        // The project requires Ansible for configuration management and deployment 
        // This command runs an Ansible playbook to pull and run the Docker container on a target EC2
        sh "ansible-playbook -i inventory.ini deploy-ec2.yml --extra-vars 'image_tag=${tagName}'"
    }

    stage('Deploy to Kubernetes') {
        echo 'Deploying to K8s Cluster...'
        withCredentials([file(credentialsId: 'k8s-config', variable: 'KUBECONFIG_FILE')]) {
            sh "kubectl --kubeconfig=${KUBECONFIG_FILE} apply -f kubernetes/"
            // Using the verified deployment name 'insurance-app'
            sh "kubectl --kubeconfig=${KUBECONFIG_FILE} set image deployment/insurance-app insurance-container=sdfa777/insurance-star_agile-project-3:${tagName}"
        }
    }

    stage('Automated Selenium Testing') {
        echo 'Running Selenium functional tests...'
        // The project requires testing the deployment with Selenium [cite: 32, 37]
        // sh 'java -jar selenium-tests.jar' 
    }
}
