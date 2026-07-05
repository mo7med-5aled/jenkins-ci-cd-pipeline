pipeline {
    agent any

    tools {
        maven 'MAVEN3.9'
        jdk 'JDK17'
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        timestamps()
        skipDefaultCheckout()
        buildDiscarder(logRotator(
            numToKeepStr: '20',
            artifactNumToKeepStr: '10'
        ))
    }

    environment {
        MVN_CMD        = 'mvn -B'
        SONAR_SCANNER  = tool('sonar8.0')
        PROJECT_KEY    = 'vprofile'
        PROJECT_NAME   = 'vprofile'
    }

    stages {

        stage('Checkout') {
            steps {
                retry(3) {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: '*/atom']],
                        userRemoteConfigs: [[
                            url: 'https://github.com/hkhcoder/vprofile-project.git'
                        ]],
                        extensions: [
                            [$class: 'CloneOption',
                                shallow: true,
                                depth: 1,
                                noTags: true,
                                timeout: 20
                            ],
                            [$class: 'CleanBeforeCheckout']
                        ]
                    ])
                }
            }
        }

        stage('Build & Unit Test') {
            steps {
                sh "${env.MVN_CMD} clean verify"
            }

            post {
                always {
                    junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Checkstyle Analysis') {
            steps {
                sh "${env.MVN_CMD} checkstyle:checkstyle"
            }

            post {
                always {
                    script {
                        if (fileExists('target/checkstyle-result.xml')) {
                            recordIssues(
                                enabledForFailure: true,
                                tools: [
                                    checkStyle(
                                        id: 'checkstyle',
                                        name: 'Checkstyle',
                                        pattern: 'target/checkstyle-result.xml'
                                    )
                                ]
                            )
                        } else {
                            echo "No Checkstyle XML report found"
                        }
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh """
                        ${env.SONAR_SCANNER}/bin/sonar-scanner \
                        -Dsonar.projectKey=${env.PROJECT_KEY} \
                        -Dsonar.projectName=${env.PROJECT_NAME} \
                        -Dsonar.sources=src/main/java \
                        -Dsonar.tests=src/test/java \
                        -Dsonar.java.binaries=target/classes \
                        -Dsonar.junit.reportPaths=target/surefire-reports \
                        -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    script {
                        def qg = waitForQualityGate()
                        echo "Quality Gate status: ${qg.status}"

                        if (qg.status != 'OK') {
                            unstable("Quality Gate failed with status: ${qg.status}")
                        }
                    }
                }
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.war', fingerprint: true
            }
        }
    }

    post {
        success {
            echo "Pipeline SUCCESS 🚀"
        }

        unstable {
            echo "Pipeline UNSTABLE ⚠️"
        }

        failure {
            echo "Pipeline FAILED ❌"
        }

        always {
            cleanWs deleteDirs: true, notFailBuild: true
        }
    }
}