pipeline {
    agent any
    tools {
        jdk "jdk17"
        maven "maven3.9.9"
    }
    environment {
        SNAP_REPO     = 'vprofile-snapshot'
        RELEASE_REPO  = 'vprofile-release'
        CENTRAL_REPO  = 'vpro-maven-central'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUSIP       = '192.168.0.68'
        NEXUSPORT     = '8081'
        NEXUSLOGIN    = 'nexuslogin'  // make sure this is the Jenkins credential ID
        SONARSCANNER  = 'sonar-scanner'
        SONARSERVER   = 'sonar-server'
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn -s settings.xml -DskipTests clean install'
            }
            post {
                success {
                    echo "Now Archiving"
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }

        stage('Checkstyle') {
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
                    sh '''
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=vprofile \
                        -Dsonar.projectName=vprofile \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=src/ \
                        -Dsonar.java.binaries=target/classes \
                        -Dsonar.junit.reportPaths=target/surefire-reports \
                        -Dsonar.jacoco.reportPaths=target/jacoco.exec \
                        -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
                    '''
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

        stage("Upload Artifact") {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                    groupId: 'Devops',
                    version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                    repository: "${RELEASE_REPO}",
                    credentialsId: "${NEXUSLOGIN}",  // fixed variable
                    artifacts: [[
                        artifactId: 'Sumit-App',
                        classifier: '',
                        file: 'target/vprofile-v2.war',
                        type: 'war'
                    ]]
                )
            }
        }
    }
    post {
        always {
            echo 'Slack Notifications.'
            slackSend channel: '#jenkinscicd',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
}
