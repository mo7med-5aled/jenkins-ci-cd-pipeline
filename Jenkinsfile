pipeline {
    agent any

    tools {
        maven "MAVEN3.9"
        jdk "JDK17"
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        timestamps()
        ansiColor('xterm')
        skipDefaultCheckout()
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'atom', url: 'https://github.com/hkhcoder/vprofile-project.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn -B clean package -DskipTests'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'mvn -B test'
            }
            post {
                always {
                    junit testResults: '**/target/surefire-reports/*.xml', allowEmptyResults: true
                }
            }
        }

        stage('Checkstyle') {
            steps {
                sh 'mvn -B checkstyle:checkstyle'
            }
            post {
                always {
                    recordIssues enabledForFailure: true, tools: [checkStyle(pattern: '**/target/checkstyle-result.xml')]
                }
            }
        }

        stage("Sonar Code Analysis") {
            environment {
                scannerHome = tool 'sonar6.2'
            }
            steps {
                withSonarQubeEnv('sonarserver') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                        -Dsonar.projectName=vprofile \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=src/ \
                        -Dsonar.java.binaries=target/classes \
                        -Dsonar.junit.reportsPath=target/surefire-reports/ \
                        -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                        -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: '**/target/*.war', allowEmptyArchive: true
            cleanWs deleteDirs: true, notFailBuild: true
        }
    }
}

