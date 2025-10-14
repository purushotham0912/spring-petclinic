pipeline {
    agent any

    tools {
        jdk 'JDK25'   // Your JDK configured in Jenkins
        maven 'Maven' // Your Maven configured in Jenkins
    }

    environment {
        GIT_REPO   = 'https://github.com/purushotham0912/spring-petclinic.git'
        GIT_BRANCH = 'main'

        TOMCAT_USER = 'ubuntu'
        TOMCAT_HOST = '13.53.62.192'
        DEPLOY_DIR  = '/opt/tomcat/webapps'
        WAR_NAME    = 'petclinic.war'

        // Path to private key on Jenkins server
        SSH_KEY = '/var/lib/jenkins/.ssh/id_rsa'
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
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Pre-Deploy Check') {
            steps {
                // Test SSH connectivity
                sh "ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ${TOMCAT_USER}@${TOMCAT_HOST} 'echo SSH OK'"

                // Check if WAR file exists locally
                script {
                    def warExists = sh(
                        script: "ls spring-petclinic/target/${WAR_NAME} || true", 
                        returnStdout: true
                    ).trim()
                    if (!warExists) {
                        error "WAR file not found! Build failed."
                    }
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                // Remove old WAR and exploded folder
                sh """
                    ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ${TOMCAT_USER}@${TOMCAT_HOST} '
                        rm -f ${DEPLOY_DIR}/${WAR_NAME} && 
                        rm -rf ${DEPLOY_DIR}/${WAR_NAME.replace(".war","")}
                    '
                """

                // Copy new WAR
                sh "scp -i ${SSH_KEY} -o StrictHostKeyChecking=no spring-petclinic/target/${WAR_NAME} ${TOMCAT_USER}@${TOMCAT_HOST}:${DEPLOY_DIR}/"

                // Restart Tomcat
                sh "ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ${TOMCAT_USER}@${TOMCAT_HOST} 'sudo systemctl restart tomcat.service'"
            }
        }
    }

    post {
        success {
            echo "✅ Deployment completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs for details."
        }
    }
}
