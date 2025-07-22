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
                slackSend channel: 'all-my-work', color: 'good', message: "Build started: ${env.JOB_NAME}#${env.BUILD_NUMBER} - Maven build in progress"
            }
        }

        stage("Maven deploy") {
            steps {
                sh "${MAVEN_HOME}/bin/mvn clean deploy"
                slackSend channel: 'all-my-work', color: 'good', message: "Build progress: ${env.JOB_NAME}#${env.BUILD_NUMBER} - Maven deploy complete"
            }
        }

        stage("Stop the Tomcat") {
            steps {
                sshagent(['Tomcat_server']) {
                    echo "Stopping Tomcat..."
                    sh "ssh -o StrictHostKeyChecking=no ec2-user@16.171.134.210 'sudo systemctl stop tomcat'"
                }
                slackSend channel: 'all-my-work', color: 'warning', message: "Stopping Tomcat on remote server"
            }
        }

        stage("Deploy WAR file to Tomcat") {
            steps {
                sshagent(['Tomcat_server']) {
                    echo "Deploying WAR file..."
                    sh "scp -o StrictHostKeyChecking=no target/student-reg-webapp.war ec2-user@16.171.134.210:/opt/tomcat/webapps/"
                }
                slackSend channel: 'all-my-work', color: 'good', message: "WAR file deployed successfully"
            }
        }

        stage("Start the Tomcat") {
            steps {
                sshagent(['Tomcat_server']) {
                    echo "Starting Tomcat..."
                    sh "ssh -o StrictHostKeyChecking=no ec2-user@16.171.134.210 'sudo systemctl start tomcat'"
                }
                slackSend channel: 'all-my-work', color: 'good', message: "Tomcat restarted successfully"
            }
        }
    }

    post {
        success {
            script {
                sendEmail(
                    "${env.JOB_NAME} - Build #${env.BUILD_NUMBER} - SUCCESS",
                    """Hi Team,<br><br>
                    The build was successful.<br>
                    View the logs: <a href='${env.BUILD_URL}'>Build Console Output</a><br><br>
                    Regards,<br>
                    Jenkins Pipeline"""
                )
                slackSend channel: 'all-my-work', color: 'good', message: "Build SUCCESS: ${env.JOB_NAME}#${env.BUILD_NUMBER}"
            }
        }
        failure {
            script {
                sendEmail(
                    "${env.JOB_NAME} - Build #${env.BUILD_NUMBER} - FAILURE",
                    """Hi Team,<br><br>
                    The build has failed.<br>
                    Check logs: <a href='${env.BUILD_URL}'>Build Console Output</a><br><br>
                    Regards,<br>
                    Jenkins Pipeline"""
                )
                slackSend channel: 'all-my-work', color: 'danger', message: "Build FAILURE: ${env.JOB_NAME}#${env.BUILD_NUMBER}"
            }
        }
    }
}

def sendEmail(String subject, String body) {
    emailext(
        subject: subject,
        body: body,
        to: 'Navya.d-shetty@capgemini.com',
        mimeType: 'text/html'
    )
}
