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

    stage('Upload to Nexus') {
      steps{
        nexusArtifactUploader artifacts: [[artifactId: 'SimpleWebApplication\'', classifier: '', file: 'target/SimpleWebApplication.war', type: 'war']], credentialsId: 'nexus-cred', groupId: 'com.maven', nexusUrl: '172.31.47.166:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots', version: '1.0.0-SNAPSHOT'
      }
    }
        stage('Deploy to Dev') {
    agent {
        label 'tomcat'
    }
    steps {
        withCredentials([usernamePassword(
            credentialsId: 'nexus-cred',
            usernameVariable: 'NEXUS_USER',
            passwordVariable: 'NEXUS_PASS'
        )]) {
            sh '''
                # Stop Tomcat
                bash /home/tomcat/tomcatserver/bin/shutdown.sh

                # Backup old WAR
                mv /home/tomcat/tomcatserver/webapps/SimpleWebApplication.war \
                   /home/tomcat/tomcatserver/webapps/SimpleWebApplication.war.bak || true

                # Download latest artifact from Nexus
                curl -fL -u "$NEXUS_USER:$NEXUS_PASS" \
                "http://172.31.44.91:8081/service/rest/v1/search/assets/download?sort=version&repository=maven-snapshots&maven.groupId=com.maven&maven.artifactId=SimpleWebApplication&maven.baseVersion=1.0.1-SNAPSHOT&maven.extension=war" \
                -o /tmp/SimpleWebApplication.war

                # Deploy WAR
                cp /tmp/SimpleWebApplication.war /home/tomcat/tomcatserver/webapps/

                # Start Tomcat
                bash /home/tomcat/tomcatserver/bin/startup.sh
            '''
        }
    }
}
}
}

