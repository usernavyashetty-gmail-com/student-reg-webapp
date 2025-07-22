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

    stage("Deploy war file to tomcat") {
        sshagent(['Tomcat_server']) {
            sh "scp -o StrictHostKeyChecking=no target/student-reg-webapp.war ec2-user@16.171.134.210:/opt/tomcat/webapps/"
        }
    }
}
