pipeline {
    agent any

    tools {
        jdk 'JDK17'     // Must match Jenkins Global Tool Configuration
        maven 'Maven'   // Must match Jenkins Global Tool Configuration
    }

    environment {
        // Java setup
        JAVA_HOME = '/usr/lib/jvm/java-17-openjdk-amd64'
        PATH      = "${JAVA_HOME}/bin:${env.PATH}"

        // Git setup
        GIT_REPO   = 'https://github.com/purushotham0912/spring-petclinic.git'
        GIT_BRANCH = 'main'

        // Tomcat deployment
        TOMCAT_USER = 'ubuntu'
        TOMCAT_HOST = '13.53.62.192'
        DEPLOY_DIR  = '/opt/tomcat/webapps'
    }

    stages {

        stage('Checkout') {
            steps {
                // Avoid workspace issues with spaces
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

        stage('Deploy to Tomcat') {
            steps {
                dir('spring-petclinic') {
                    script {
                        // Get the WAR file
                        def warFile = sh(script: "ls target/*.war | head -n 1", returnStdout: true).trim()
                        echo "Deploying WAR: ${warFile}"

                        // Copy WAR to Tomcat server
                        sh "scp ${warFile} ${TOMCAT_USER}@${TOMCAT_HOST}:${DEPLOY_DIR}/petclinic.war"

                        // Restart Tomcat (requires NOPASSWD sudo)
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
