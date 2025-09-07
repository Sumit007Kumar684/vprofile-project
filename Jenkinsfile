pipeline {
    agent any
    tools {
        jdk "jdk17"
        maven "maven3.9.9" ///maven
    }
    stages{
        stage('Build'){
            steps{
                sh 'mvn -DskipTests install'
            }
        }
    }
}