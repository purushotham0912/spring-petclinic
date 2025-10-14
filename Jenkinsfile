pipeline {
    agent any

    environment {
        GIT_REPO   = 'https://github.com/purushotham0912/spring-petclinic.git'
        GIT_BRANCH = 'main'

        TOMCAT_USER = 'ubuntu'
        TOMCAT_HOST = '13.53.62.192'
        DEPLOY_DIR  = '/opt/tomcat/webapps'
    }

    stages {
        stage('Checkout') {
            steps {
                dir('spring-petclinic') {
                    git branch: "${GIT_BRANCH}", url: "${GIT_REPO}"
                }
            }
        }

        stage('Build') {
            steps {
                dir('spring-petclinic') {
                    sh '''
                        export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
                        export PATH=$JAVA_HOME/bin:$PATH
                        mvn clean package -DskipTests
                    '''
                }
            }
        }

        stage('Pre-Deploy Check') {
            steps {
                script {
                    sh "ssh -o BatchMode=yes ${TOMCAT_USER}@${TOMCAT_HOST} 'echo SSH OK'"
                    def warExists = sh(script: "ls target/*.war || true", returnStdout: true).trim()
                    if (!warExists) {
                        error "WAR file not found!"
                    }
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                dir('spring-petclinic') {
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

    post {
        success { echo "Deployment completed successfully!" }
        failure { echo "Pipeline failed. Check logs." }
    }
}
