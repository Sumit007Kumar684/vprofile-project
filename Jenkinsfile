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
        NEXUSIP = '192.168.0.68'
        NEXUSPORT = '8081'
        NEXUSLOGIN = 'nexuslogin' 
        SONARSCANNER =  'sonar-scanner'
        SONARSERVER = 'sonar-server'
    }
    stages{
        stage('Build'){
            steps{
                sh 'mvn -s settings.xml -DskipTests install' //settings.xml
            }
            post {
                success {
                    echo "Now Archiving"
                    archiveArtifacts artifacts:'**/*.war'
                }
            }
        }
        stage('Checkstyle'){
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }
        stage('Sonar Analysis') {
            environment {
                scannerHome = tool "${SONARSCANNER}"
            }
            steps {
               withSonarQubeEnv("${SONARSERVER}") {
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
              }
            }
        }
        stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
              }
            }
        }
    }
}

