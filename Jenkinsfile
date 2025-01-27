node {
    def mavenHome
    def mavenCMD
    def docker
    def dockerCMD
    def tagName = "3.0"

    stage('Prepare Environment') {
        echo 'Initializing variables...'
        mavenHome = tool name: 'maven', type: 'maven'
        mavenCMD = "${mavenHome}/bin/mvn"
        docker = tool name: 'docker', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
        dockerCMD = "${docker}/bin/docker"
    }

    stage('Git Code Checkout') {
        try {
            echo 'Checking out code from Git repository...'
            git 'https://github.com/Undertaker69cyprus/adithya-insurance-project.git'
        } catch (Exception e) {
            echo 'Error during Git checkout.'
            currentBuild.result = "FAILURE"
            emailext body: '''Dear Team,
            The Jenkins job ${JOB_NAME} has failed during the Git checkout stage. Please review the logs:
            ${BUILD_URL}''', subject: 'Job Failure: ${JOB_NAME} Build ${BUILD_NUMBER}', to: 'adithyamr20@gmail.com'
            error("Terminating pipeline due to Git checkout failure.")
        }
    }

    stage('Build the Application') {
        echo "Cleaning, compiling, testing, and packaging the application..."
        sh "${mavenCMD} clean package"
    }

    stage('Publish Test Reports') {
        publishHTML([
            allowMissing: false, 
            alwaysLinkToLastBuild: false, 
            keepAll: false, 
            reportDir: 'target/surefire-reports', 
            reportFiles: 'index.html', 
            reportName: 'Test Report', 
            reportTitles: '', 
            useWrapperFileDirectly: true
        ])
    }

    stage('Containerize the Application') {
        echo 'Building Docker image...'
        sh "${dockerCMD} build -t addi2/insure-me1:${tagName} ."
    }

    stage('Push to DockerHub') {
        echo 'Pushing the Docker image to DockerHub...'
        withCredentials([usernamePassword(credentialsId: 'dock-password', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
            sh "${dockerCMD} login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}"
            sh "${dockerCMD} push addi2/insure-me1:${tagName}"
        }
    }

    stage('Deploy to Test Server') {
        echo 'Deploying using Ansible...'
        ansiblePlaybook(
            become: true, 
            credentialsId: 'ansible-key', 
            disableHostKeyChecking: true, 
            installation: 'ansible', 
            inventory: '/etc/ansible/hosts', 
            playbook: 'ansible-playbook.yml'
        )
    }
}
