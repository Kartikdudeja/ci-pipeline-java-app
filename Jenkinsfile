def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger'
]

pipeline {
    agent any

    tools {
        maven "MAVEN3"
        jdk "OracleJDK8"
    }

    environment {
        SNAP_REPO = ''
        NEXUS_USER = ''
        NEXUS_PASS = ''
        RELEASE_REPO = ''
        CENTRAL_REPO = ''
        NEXUSIP = ''
        NEXUSPORT = ''
        NEXUS_GRP_REPO = ''
        NEXUS_LOGIN = ''
        SONARSERVER = ''
        SONARSCANNER = ''
        PROJECT_KEY = ''
        PROJECT_NAME = ''
        VERSION = ''
        PROJSRC_DIR = ''
        JAVA_BINARIES = ''
        JUNIT_PATH = ''
        JACOCO_EXEC_PATH = ''
        CHECKSTYPE_FILE_PATH = ''
        SLACK_CHANNEL = '#cicd'
    }

    stages {
        stage ('Build') {
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }

            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/*.war'
                }
            }

        }

        stage ('Test') {
            steps {
                sh 'mvn -s settings.xml test'
            }
        }

        // Code Analysis
        stage ('Checkstyle Analysis') {
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }

        stage ('Sonar Analysis') {
            
            environment {
                scannerHome = tool "${SONARSCANNER}"
            }

            steps {
                withSonarQubeEnv("${SONARSERVER}") {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=${PROJECT_KEY} \
                    -Dsonar.projectName=${PROJECT_NAME} \
                    -Dsonar.projectVersion=${VERSION} \
                    -Dsonar.sources=${PROJECT_SRC_DIR} \
                    -Dsonar.java.binaries=${JAVA_BINARIES} \
                    -Dsonar.junit.reportsPath=${JUNIT_PATH} \
                    -Dsonar.jacoco.reportsPath=${JACOCO_EXEC_PATH} \
                    -Dsonar.java.checkstyle.reportPaths=${CHECKSTYPE_FILE_PATH}'''
                }
            }
        }

        stage ('Quality Gate') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage ('Upload Artifact') {

            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                    groupId: 'QA',
                    version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                    repository: "${RELEASE_REPO}",
                    credentialsId: "${NEXUS_LOGIN}",
                    artifacts: [
                        [ 
                            artifactId: "${PROJECT_KEY}",
                            classifier: '',
                            file: "target/${PROJECT_NAME}-v${VERSION}.war",
                            type: 'war'
                        ]
                    ]
                )                
            }
        }
    }

    post {
        always {
            echo 'Slack Notification'
            slackSend channel: "${SLACK_CHANNEL}",
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \nMore Info at: ${env.BUILD_URL}"
        }
    }
}
