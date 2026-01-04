node {
    def mavenHome
    def mavenCMD
    def dockerCMD = "docker"
    def tagName = "3.0"
    def monitoringIP = "10.0.1.107" // <--- UPDATE THIS TO YOUR MONITORING EC2 IP

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
        withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials-id', 
                                          usernameVariable: 'U', 
                                          passwordVariable: 'P')]) {
            sh 'echo $P | docker login -u $U --password-stdin'
            sh "docker push sdfa777/insurance-star_agile-project-3:${tagName}"
        }
    }

    stage('Deploy to EC2 (via Monitoring Node)') {
        echo 'Triggering Ansible on Monitoring EC2...'
        withCredentials([file(credentialsId: 'monitoring-pem-key', variable: 'PEM')]) {
            sh """
                chmod 400 ${PEM}
                # 1. We use the full path /usr/bin/ansible-playbook
                # 2. We use the ansible user
                # 3. We point to the correct file locations we found earlier
                ssh -o StrictHostKeyChecking=no -i ${PEM} ansible@10.0.1.107 \
                "/usr/bin/ansible-playbook -i /home/ansible/ansible/inventory.ini /home/ansible/ansible/playbooks/deploy-ec2.yml --extra-vars 'image_tag=${tagName}'"
            """
        }
    }

    stage('Deploy to Kubernetes') {
        echo 'Deploying to K8s...'
        withCredentials([file(credentialsId: 'k8s-config', variable: 'KUBECONFIG_FILE')]) {
            sh 'kubectl --kubeconfig=$KUBECONFIG_FILE apply -f kubernetes/'
            sh "kubectl --kubeconfig=$KUBECONFIG_FILE set image deployment/insurance-app insurance-container=sdfa777/insurance-star_agile-project-3:${tagName}"
        }
    }
}
