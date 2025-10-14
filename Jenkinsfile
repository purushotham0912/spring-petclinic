pipeline {
    agent any
    tools {
        jdk 'JDK17'
        maven 'Maven3'
    }

    environment {
        TOMCAT_USER = 'ubuntu'
        TOMCAT_HOST = '13.53.62.192'
        DEPLOY_DIR  = '/opt/tomcat/webapps'
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/purushotham0912/spring-petclinic.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    def warFile = sh(script: "ls target/*.war | head -n 1", returnStdout: true).trim()
                    echo "Deploying WAR: ${warFile}"
                    sh "scp ${warFile} ${TOMCAT_USER}@${TOMCAT_HOST}:${DEPLOY_DIR}/petclinic.war"
                    sh "ssh ${TOMCAT_USER}@${TOMCAT_HOST} 'sudo systemctl restart tomcat'"
                }
            }
        }
    }
}
