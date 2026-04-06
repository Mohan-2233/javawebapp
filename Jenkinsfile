pipeline {
    agent {
        label ('java')
    }
    stages {
        stage ('Checkout SCM') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/main']],
                    extensions: [],
                    userRemoteConfigs: [[
                        credentialsId: 'github-cred',
                        url: 'https://github.com/Mohan-2233/javawebapp.git'
                    ]]
                )
            }
        }
        stage ('Build and Test') {
            steps {
                sh 'mvn clean verify'
            }
        }
        stage ('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'sonarscanner'
                    withSonarQubeEnv('sonar') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                              -Dsonar.projectKey=javawebapp \
                              -Dsonar.projectName=javawebapp \
                              -Dsonar.projectVersion=${BUILD_NUMBER} \
                                                            -Dsonar.scm.provider=git \
                              -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml \
                              -Dsonar.sources=src/main/java,src/main/webapp \
                              -Dsonar.tests=src/test/java \
                              -Dsonar.java.binaries=target/classes \
                              -Dsonar.java.test.binaries=target/test-classes \
                              -Dsonar.junit.reportPaths=target/surefire-reports
                            """
                    }
                }
            }
        }
        stage("Sonar Quality Gate Check") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    script {
                        def qualityGate = waitForQualityGate()
                        if (qualityGate.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qualityGate.status}"
                        }
                    }
                } // End of timeout
            }
        }
    }
}
