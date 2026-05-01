pipeline {
    agent any

    stages {
    
        stage('checkout') {
            steps {
                echo 'git checkout stage'
                git branch: 'main', url: 'https://github.com/jadalaramani/slack_tomcat_jenkins.git'
            }
        }

        stage('build') {
            steps {
                echo 'Building with maven'
                sh 'mvn clean install'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying to tomcat'
                deploy adapters: [tomcat9(
                    credentialsId: 'tomcat',
                    url: 'http://13.217.10.33:8081/'
                )],
                contextPath: 'slack',
                war: '**/*.war'
            }
        }        
    }

  post {
    success {
        withCredentials([string(credentialsId: 'slack-webhook', variable: 'SLACK_URL')]) {
            sh """
            curl -X POST -H "Content-type: application/json" \
            --data "{\\"text\\":\\"✅ SUCCESS: ${JOB_NAME} Build #${BUILD_NUMBER} deployed successfully 🚀\\nURL: ${BUILD_URL}\\"}" \
            $SLACK_URL
            """
        }
    }

    failure {
        withCredentials([string(credentialsId: 'slack-webhook', variable: 'SLACK_URL')]) {
            sh """
            curl -X POST -H "Content-type: application/json" \
            --data "{\\"text\\":\\"❌ FAILURE: ${JOB_NAME} Build #${BUILD_NUMBER} failed 💥\\nURL: ${BUILD_URL}\\"}" \
            $SLACK_URL
            """
        }
    }
}
}
