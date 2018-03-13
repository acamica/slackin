pipeline {
    agent any
    tools {
        nodejs 'Node 8.9.1'
    }
    parameters {
        booleanParam(name: 'RELEASE', defaultValue: false, description: 'Generar un release?')
    }

    environment {
        GITHUB_TOKEN  = credentials('acamica-release-token')
        REGISTRY      = credentials('acamica-registry-password')
        CI            = 1
    }
    stages {
        stage('Install dependencies') {
            steps {
                sh 'npm doctor'
                sh 'npm install'
            }
        }

        stage('Semantic release') {
            when { expression { return params.RELEASE } }

            steps {
                sh 'npm run semantic-release'
            }
        }

        stage('Build Docker images for Production') {
            steps {
                // TODO: check if we can infer production environment from config
                sh 'PRODUCTION=1 ./docker-build.sh ${JOB_NAME} jenkins-${BUILD_NUMBER}'
                sh 'docker login -u ${REGISTRY_USR} -p ${REGISTRY_PSW} registry.acamica.com' //TODO: replace with folder credentials
                sh './docker-push.sh ${JOB_NAME} jenkins-${BUILD_NUMBER}'
            }
        }
    }

    post {
        success {
            slackSend channel: env.SLACK_SUCC_CHANNEL, color: 'good', message: "Build finished successfully : ${env.JOB_NAME} - ${env.BUILD_NUMBER} (<${env.JOB_URL}|Open>)"
        }
        failure {
            slackSend channel: env.SLACK_ERR_CHANNEL, color: '#FF0000', message: "Build failed: ${env.JOB_NAME} - ${env.BUILD_NUMBER} (<${env.JOB_URL}|Open>)"
        }
    }
}