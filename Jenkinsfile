pipeline {
    agent any

    tools {
        maven "maven"
        sonarScanner "sonar_scanner"
    }

    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "54.85.150.44:8082"
        NEXUS_REPOSITORY = "simplecustomerapp"
        NEXUS_CREDENTIAL_ID = "nexus-creds"
    }

    stages {
        stage("Clone Code") {
            steps {
                git 'https://github.com/vaeshwar/sabear_simplecutomerapp.git'
            }
        }

        stage("Maven Build") {
            steps {
                sh 'mvn -Dmaven.test.failure.ignore=true clean install'
            }
        }

        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh 'sonar-scanner'
                }
            }
        }

        stage("Publish to Nexus") {
            steps {
                script {
                    def pom = readMavenPom file: 'pom.xml'
                    def artifactFiles = findFiles(glob: "target/*.${pom.packaging}")
                    if (artifactFiles.length == 0) {
                        error "No artifact found in target/"
                    }

                    def artifact = artifactFiles[0].path
                    echo "Artifact found: ${artifact}"

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
                             file: artifact,
                             type: pom.packaging],
                            [artifactId: pom.artifactId,
                             classifier: '',
                             file: 'pom.xml',
                             type: 'pom']
                        ]
                    )
                }
            }
        }
    }
}
