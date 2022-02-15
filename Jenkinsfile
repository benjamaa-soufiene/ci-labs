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
                stage('CheckStyle') {
                    agent {
                        docker {
                            image 'maven:3.6.0-jdk-8-alpine'
                            args '-v /root/.m2/repository:/root/.m2/repository'
                            reuseNode true
                        }
                    }
                    steps {
                        sh ' mvn checkstyle:checkstyle'
                    }
                    post {
                        always {
                            recordIssues enabledForFailure: true, tool: checkStyle()
                        }
                    }
                }
            }
        }
        stage('Unit Tests') {
            when {
                anyOf { branch 'master'; branch 'develop' }
            }
            agent {
                docker {
                    image 'maven:3.6.0-jdk-8-alpine'
                    args '-v /root/.m2/repository:/root/.m2/repository'
                    reuseNode true
                }
            }
            steps {
                sh ' mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/**/*.xml'
                }
            }
        }
        stage('Integration Tests') {
            when {
                anyOf { branch 'master'; branch 'develop' }
            }
            agent {
                docker {
                    image 'maven:3.6.0-jdk-8-alpine'
                    args '-v /root/.m2/repository:/root/.m2/repository'
                    reuseNode true
                }
            }
            steps {
                sh 'mvn verify -Dsurefire.skip=true'
            }
            post {
                always {
                    junit 'target/failsafe-reports/**/*.xml'
                }
                success {
                    stash(name: 'artifact', includes: 'target/*.jar')
                    stash(name: 'pom', includes: 'pom.xml')
                    archiveArtifacts 'target/*.jar'
                }
            }
        }
        stage('Code Quality Analysis') {
            parallel {
                stage('PMD') {
                    agent {
                        docker {
                            image 'maven:3.6.0-jdk-8-alpine'
                            args '-v /root/.m2/repository:/root/.m2/repository'
                            reuseNode true
                        }
                    }
                    steps {
                        sh ' mvn pmd:pmd'
                    }
                    post {
                        always {
                            recordIssues enabledForFailure: true, tool: pmdParser(pattern: '**/target/pmd.xml')
                        }
                    }
                }
                stage('Findbugs') {
                    agent {
                        docker {
                            image 'maven:3.6.0-jdk-8-alpine'
                            args '-v /root/.m2/repository:/root/.m2/repository'
                            reuseNode true
                        }
                    }
                    steps {
                        sh ' mvn findbugs:findbugs'
                    }
                    post {
                        always {
                            recordIssues enabledForFailure: true, tool: spotBugs(pattern: '**/target/findbugsXml.xml')
                        }
                    }
                }
                stage('JavaDoc') {
                    agent {
                        docker {
                            image 'maven:3.6.0-jdk-8-alpine'
                            args '-v /root/.m2/repository:/root/.m2/repository'
                            reuseNode true
                        }
                    }
                    steps {
                        sh ' mvn javadoc:javadoc'
                        step([$class: 'JavadocArchiver', javadocDir: './target/site/apidocs', keepAll: 'true'])
                    }
                    post {
                        always {
                            recordIssues enabledForFailure: true, tools: [mavenConsole(), java(), javaDoc()]
                        }
                    }
                }
            }
        }
    }
}
