pipeline {
    agent any

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        jdk "jdk17"
        maven "maven3"
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "nexus:8081"
        NEXUS_REPOSITORY = "backend-nexus-repo"
        NEXUS_CREDENTIAL_ID = "nexus-user-credentials"
    }

    stages {
        stage('git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Anas4444/backend.git'
            }
        }

        stage('Code Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('SONARQUBE-ANALYSIS') {
            steps() {
                withSonarQubeEnv('sonarqube-server') {
                    //sh "mvn sonar:sonar"
                    sh '${SCANNER_HOME}/bin/sonar-scanner --version'
                    /*sh ''' ${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectName=backend \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=backend '''*/
                }
            }
        }

        stage('Build') {
            steps() {
                sh "mvn clean package -DskipTests=true"
            }
        }

        stage('Publish to Nexus Repository Manager') {
            steps() {
                script {
                    pom = readMavenPom file: "pom.xml"
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    artifactPath = filesByGlob[0].path
                    artifactExists = fileExists artifactPath
                    nexusArtifactUploader(
                        nexusVersion: NEXUS_VERSION,
                        protocol: NEXUS_PROTOCOL,
                        nexusUrl: NEXUS_URL,
                        groupId: pom.groupId,
                        version: pom.version,
                        repository: NEXUS_REPOSITORY,
                        credentialsId: NEXUS_CREDENTIAL_ID,
                        artifacts: [
                            [artifactId: pom.artifactId,
                            classifier: '',
                            file: artifactPath,
                            type: pom.packaging],
                            [artifactId: pom.artifactId,
                            classifier: '',
                            file: "pom.xml",
                            type: "pom"]
                        ]
                    )
                }
            }
        }
    }
}