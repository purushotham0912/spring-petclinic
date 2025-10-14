pipeline {
    agent any

    tools {
        jdk 'JDK17'   // Name configured in Jenkins global tools
        maven 'Maven' // Name configured in Jenkins global tools
    }

    environment {
        GIT_REPO   = 'https://github.com/purushotham0912/spring-petclinic.git'
        GIT_BRANCH = 'main'

        TOMCAT_USER = 'ubuntu'
        TOMCAT_HOST = '13.53.62.192'          // Replace with your Tomcat EC2 public IP
        DEPLOY_DIR  = '/opt/tomcat/webapps'
        WAR_NAME    = 'petclinic.war'
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
                script {
                    // Test SSH connectivity
                    sh "ssh -o BatchMode=yes ${TOMCAT_USER}@${TOMCAT_HOST} 'echo SSH OK'"

                    // Check if WAR file exists
                    def warExists = sh(script: "ls target/*.war || true", returnStdout: true).trim()
                    if (!warExists) {
                        error "WAR file not found! Build failed."
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

                        // Copy WAR to Tomcat
                        sh "scp ${warFile} ${TOMCAT_USER}@${TOMCAT_HOST}:${DEPLOY_DIR}/${WAR_NAME}"

                        // Restart Tomcat
                        sh "ssh ${TOMCAT_USER}@${TOMCAT_HOST} 'sudo systemctl restart tomcat'"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Deployment completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check logs for details."
        }
    }
}
