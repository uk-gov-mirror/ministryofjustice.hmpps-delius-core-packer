def verify_image(filename) {
    wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'XTerm']) {
        sh '''
        #!/usr/env/bin bash
        docker run --rm \
        -e BRANCH_NAME \
        -e TARGET_ENV \
        -e ARTIFACT_BUCKET \
        -v `pwd`:/home/tools/data \
        mojdigitalstudio/hmpps-packer-builder \
        bash -c 'ansible-galaxy install -r ansible/requirements.yml; USER=`whoami` packer validate ''' + filename + "'"
    }
}

def build_image(filename) {
    wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'XTerm']) {
        sh '''
        #!/usr/env/bin bash
        docker run --rm \
        -e BRANCH_NAME \
        -e TARGET_ENV \
        -e ARTIFACT_BUCKET \
        -v `pwd`:/home/tools/data \
        mojdigitalstudio/hmpps-packer-builder \
        bash -c 'ansible-galaxy install -r ansible/requirements.yml; USER=`whoami` packer build ''' + filename + "'"
    }
}

pipeline {
    agent { label "jenkins_slave"}

    stages {
        stage ('Notify build started') {
            steps {
                slackSend(message: "Build Started - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL.replace(':8080','')}|Open>)")
            }
        }

        stage('Verify Delius-Core AMIS') {
            parallel {
                stage('Verify Delius-Core Weblogic') { steps { script {verify_image('weblogic.json')}}}
            }
        }

        stage('Build Delius-Core AMIS') {
            parallel {
                stage('Build Delius-Core Weblogic') { steps { script {build_image('weblogic.json')}}}
            }
        }
    }

    post {
        success {
            slackSend(message: "Build completed - ${env.JOB_NAME} ${env.BUILD_NUMBER}", color: 'good')
        }
        failure {
            slackSend(message: "Build failed - ${env.JOB_NAME} ${env.BUILD_NUMBER}", color: 'danger')
        }
    }
}