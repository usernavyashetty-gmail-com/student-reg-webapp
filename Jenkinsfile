pipeline {
    agent any

    tools {
        maven 'maven-3.9.1'
    }

    environment {
        MAVEN_HOME = tool(name: 'maven-3.9.1', type: 'maven')
    }

    stages {
        stage("Git clone") {
            steps {
                git branch: 'main', credentialsId: 'githubcred', url: 'https://github.com/usernavyashetty-gmail-com/student-reg-webapp.git'
            }
        }

        stage("Maven build") {
            steps {
                sh "${MAVEN_HOME}/bin/mvn clean verify sonar:sonar"
            }
        }

        stage("Maven deploy") {
            steps {
                sh "${MAVEN_HOME}/bin/mvn clean deploy"
            }
        }

        stage("Stop the Tomcat") {
            steps {
                sshagent(['Tomcat_server']) {
                    echo "Stopping Tomcat..."
                    sh "ssh -o StrictHostKeyChecking=no ec2-user@16.171.134.210 'sudo systemctl stop tomcat'"
                }
            }
        }

        stage("Deploy WAR file to Tomcat") {
            steps {
                sshagent(['Tomcat_server']) {
                    echo "Deploying WAR file..."
                    sh "scp -o StrictHostKeyChecking=no target/student-reg-webapp.war ec2-user@16.171.134.210:/opt/tomcat/webapps/"
                }
            }
        }

        stage("Start the Tomcat") {
            steps {
                sshagent(['Tomcat_server']) {
                    echo "Starting Tomcat..."
                    sh "ssh -o StrictHostKeyChecking=no ec2-user@16.171.134.210 'sudo systemctl start tomcat'"
                }
            }
        }
    }

    post {
        success {
            script {
                sendEmail("${env.JOB_NAME} - Build #${env.BUILD_NUMBER} - SUCCESS",
                          "Build Status: SUCCESS. Please check the console output at ${env.BUILD_URL} for more details.")
            }
        }
        failure {
            script {
                sendEmail("${env.JOB_NAME} - Build #${env.BUILD_NUMBER} - FAILURE",
                          "Build Status: FAILURE. Please check the console output at ${env.BUILD_URL} for more details.")
            }
        }
    }
}

def sendEmail(String subject, String body) {
    emailext(
        subject: subject,
        body: body,
        to: 'Usernavyashetty@gmail.com',
        mimeType: 'text/html'
    )
}
