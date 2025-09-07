pipeline {
    agent any
    tools {
        jdk "jdk17"
        maven "maven3.9.9" ///maven
    }
    environment {
        SNAP_REPO = 'vprofile-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'admin'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vpro-maven-central'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_IP = '192.168.0.68'
        NEXUS_PORT = '8081'
        NEXUSLOGIN = 'nexuslogin' //
    }
    stages{
        stage('Build'){
            steps{
                sh 'mvn -s settings.xml -DskipTests install'
            }
            post {
                success {
                    echo "Now Archiving"
                    archiveArtifacts artifacts:'**/*.war'
                }
            }
        }
    }
}