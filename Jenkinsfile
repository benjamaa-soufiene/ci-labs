pipeline {
    agent any
    stages {
        stage('SCM') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            parallel {
                stage('Compile') {
                    agent {
                        docker {
                            image 'maven:3.6.0-jdk-8-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh ' mvn clean compile'
                    }
                }
            }
        }
    }
}
