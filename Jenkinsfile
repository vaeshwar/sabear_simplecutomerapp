node {
    def pom

    // Tool configuration
    tools {
        maven "maven"
    }

    // Environment variables
    env.NEXUS_VERSION = "nexus3"
    env.NEXUS_PROTOCOL = "http"
    env.NEXUS_URL = "54.85.150.44:8082"
    env.NEXUS_REPOSITORY = "simplecustomerapp"
    env.NEXUS_CREDENTIAL_ID = "nexus-creds"

    try {
        stage("Clone Code") {
            git 'https://github.com/vaeshwar/sabear_simplecutomerapp.git'
        }

        stage("Maven Build") {
            sh 'mvn -Dmaven.test.failure.ignore=true clean install'
        }

        stage("SonarQube Analysis") {
            withSonarQubeEnv('sonarqube') {
                withEnv(["PATH=/opt/sonar-scanner/bin:$PATH"]) {
                    sh '''
                        mkdir -p extracted_classes
                        unzip -oq target/*.war -d extracted_classes
                        sonar-scanner -Dsonar.java.binaries=extracted_classes/WEB-INF/classes
                    '''
                }
            }
        }

        stage("Publish to Nexus") {
            script {
                pom = readMavenPom file: 'pom.xml'
                def artifactFiles = findFiles(glob: "target/*.${pom.packaging}")
                if (artifactFiles.length == 0) {
                    error "No artifact found in target/"
                }

                def artifact = artifactFiles[0].path
                echo "Artifact found: ${artifact}"

                nexusArtifactUploader(
                    nexusVersion: env.NEXUS_VERSION,
                    protocol: env.NEXUS_PROTOCOL,
                    nexusUrl: env.NEXUS_URL,
                    groupId: pom.groupId,
                    version: pom.version,
                    repository: env.NEXUS_REPOSITORY,
                    credentialsId: env.NEXUS_CREDENTIAL_ID,
                    artifacts: [
                        [
                            artifactId: pom.artifactId,
                            classifier: '',
                            file: artifact,
                            type: pom.packaging
                        ],
                        [
                            artifactId: pom.artifactId,
                            classifier: '',
                            file: 'pom.xml',
                            type: 'pom'
                        ]
                    ]
                )
            }
        }
    } catch (err) {
        currentBuild.result = 'FAILURE'
        throw err
    } finally {
        slackSend (
            channel: '#all-tech',
            color: currentBuild.result == 'SUCCESS' ? 'good' : 'danger',
            message: "Build *${env.JOB_NAME}* #${env.BUILD_NUMBER} finished with status: *${currentBuild.result}*",
            tokenCredentialId: 'slack-token'
        )
    }
}
