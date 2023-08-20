// Jenkinsfile for CI Pipeline for JAVA Application

// Color Mapping to indicated build result statuses (SUCCESS and FAILURE)
def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger'
]

pipeline {
    agent any

    // required tools
    tools {
        maven "MAVEN3"
        jdk "OracleJDK8"
    }

    // environment variable
    environment {
        // Nexus repository details
        NEXUS_USER = ''
        NEXUS_PASS = ''
        NEXUS_IP = ''
        NEXUS_PORT = ''
        NEXUS_LOGIN = ''
        RELEASE_REPO = ''
        ARTIFACT_TARGET_DIR = ''

        // SonarQube server details
        SONAR_SERVER = ''
        SONAR_SCANNER = ''

        // SonarQube CLI Argument variables
        PROJECT_KEY = ''
        PROJECT_NAME = ''
        PROJECT_VERSION = ''
        PROJECT_SRC_DIR = ''
        JAVA_BINARIES = ''
        JUNIT_PATH = ''
        JACOCO_EXEC_PATH = ''
        CHECKSTYPE_FILE_PATH = ''

        // Slack Channel for Notifications
        SLACK_CHANNEL = ''
    }

    stages {
        stage ('Build') {
            steps {
                // Maven command to build the project
                sh 'mvn -s settings.xml -DskipTests install'
            }

            post {
                success {
                    // After a successful build, archive the generated WAR files
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/*.war'
                }
            }

        }

        stage ('Test') {
            steps {
                // runs maven tests
                sh 'mvn -s settings.xml test'
            }
        }

        // Code Analysis
        stage ('Checkstyle Analysis') {
            steps {
                // runs the Checkstyle tool to check code style and conventions
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }

        // perform a deeper analysis using SonarQube
        stage ('Sonar Analysis') {
            
            environment {
                scannerHome = tool "${SONAR_SCANNER}"
            }

            steps {
                withSonarQubeEnv("${SONAR_SERVER}") {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=${PROJECT_KEY} \
                    -Dsonar.projectName=${PROJECT_NAME} \
                    -Dsonar.projectVersion=${PROJECT_VERSION} \
                    -Dsonar.sources=${PROJECT_SRC_DIR} \
                    -Dsonar.java.binaries=${JAVA_BINARIES} \
                    -Dsonar.junit.reportsPath=${JUNIT_PATH} \
                    -Dsonar.jacoco.reportsPath=${JACOCO_EXEC_PATH} \
                    -Dsonar.java.checkstyle.reportPaths=${CHECKSTYPE_FILE_PATH}'''
                }
            }
        }

        // stage waits for the SonarQube Quality Gate to pass. This gate ensures that the code quality and analysis metrics meet the defined criteria
        stage ('Quality Gate') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        // uploads the built WAR artifact to a Nexus repository
        stage ('Upload Artifact') {

            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${NEXUS_IP}:${NEXUS_PORT}",
                    groupId: 'QA',
                    version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                    repository: "${RELEASE_REPO}",
                    credentialsId: "${NEXUS_LOGIN}",
                    artifacts: [
                        [ 
                            artifactId: "${PROJECT_KEY}",
                            classifier: '',
                            file: "${ARTIFACT_TARGET_DIR}/${PROJECT_NAME}-v${VERSION}.war",
                            type: 'war'
                        ]
                    ]
                )                
            }
        }
    }

    post {
        always {
            // sends a notification to Slack
            echo 'Slack Notification'
            slackSend channel: "${SLACK_CHANNEL}",
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \nMore Info at: ${env.BUILD_URL}"
        }
    }
}
