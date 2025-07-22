node {
    def mavenhome = tool name: 'maven-3.9.1', type: 'maven'

  


    stage("Git clone") {
        git branch: 'main', credentialsId: 'githubcred', url: 'https://github.com/usernavyashetty-gmail-com/student-reg-webapp.git'
    }

    stage("Maven build") {
        sh "${mavenhome}/bin/mvn clean verify sonar:sonar"
    }

    stage("Maven deploy") {
        sh "${mavenhome}/bin/mvn clean deploy"
    }

    stage("Stop the tomcat") {
        sshagent(['Tomcat_server']) {
            echo "Stopping Tomcat..."
            sh "ssh -o StrictHostKeyChecking=no ec2-user@16.171.134.210 'sudo systemctl stop tomcat'"
        }
    }

    stage("Deploy war file to tomcat") {
        sshagent(['Tomcat_server']) {
            echo "Deploying WAR file..."
            sh "scp -o StrictHostKeyChecking=no target/student-reg-webapp.war ec2-user@16.171.134.210:/opt/tomcat/webapps/"
        }
    }

    stage("Start the tomcat") {
        sshagent(['Tomcat_server']) {
            echo "Starting Tomcat..."
            sh "ssh -o StrictHostKeyChecking=no ec2-user@16.171.134.210 'sudo systemctl start tomcat'"
        }
    }
}
