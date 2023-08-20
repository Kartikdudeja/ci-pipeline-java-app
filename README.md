# Continuous Integration (CI) Pipeline for Java Application

This repository contains a Jenkins Pipeline script written in Groovy that automates the building, testing, analysis of Java Web Application. The pipeline is designed to ensure code quality, thorough testing of the application using a series of defined stages.

## Prerequisites

- Jenkins: The pipeline script is designed to be executed within a Jenkins environment.
- Maven: The Java application is built using Maven.
- SonarQube: The SonarQube server is used for comprehensive code analysis.
- Nexus Repository Manager: The pipeline uploads built artifacts to a Nexus repository.
- Slack Integration: The pipeline sends notifications to a Slack channel.

## Pipeline Overview

The Jenkins pipeline consists of several stages, each serving a specific purpose in the CI/CD process:

1. **Build Stage:**
   - Compiles the Java application using Maven.
   - Skips running tests to accelerate the build process.
   - Archives generated WAR (web application archive) files for later use.

2. **Test Stage:**
   - Executes Maven tests to validate the application's functionality.

3. **Code Analysis Stages:**
   - **Checkstyle Analysis:** Checks code style and conventions using Checkstyle.
   - **Sonar Analysis:** Performs in-depth code analysis using SonarQube, evaluating code quality and metrics.

4. **Quality Gate Stage:**
   - Waits for the SonarQube Quality Gate to pass.
   - The Quality Gate ensures that code quality standards are met before proceeding.

5. **Upload Artifact Stage:**
   - Uploads the built WAR artifact to a Nexus repository.
   - The artifact becomes available for deployment or distribution.

6. **Notification:**
   - Sends a Slack notification after each stage completion.
   - The notification includes build status, job details, and a link to further information.

## Usage

1. Set up Jenkins:
   - Ensure Jenkins is installed and configured in your environment.
   - Install the following Plugins: `Maven Integration`, `Github Integration`, `Nexus Artifact Uploader`, `Sonarqube Scanner`, `Slack Notification` and `Build Timestamp`.
 
2. Configure Tools:
   - Configure Maven (MAVEN3) and JDK (OracleJDK8) in your Jenkins instance.
   - Create Nexus Repository
   - Create Quality Gate on Sonarqube Server
   - Add Nexus and Sonarqube Credentials under Jenkins Global Credentials

3. Configure Environment Variables:
   - Update the environment variables in the pipeline script (`Jenkinsfile`) to match your environment's settings. These variables include Nexus repository details, SonarQube server information, and Slack channel details.

4. Create a Jenkins Pipeline:
   - Create a new Jenkins pipeline job and specify the pipeline script (`Jenkinsfile`) as the pipeline definition.

5. Run the Pipeline:
   - Trigger the pipeline manually or configure it to run automatically based on your needs.

6. Monitor and Review:
   - Monitor the pipeline's progress within Jenkins.
   - Review build logs and artifacts for each stage's output.

## Notifications

- The pipeline sends Slack notifications after each stage's completion.
- The notifications indicate the current build status, job name, build number, and a link to further build details.

## Further Customization

- You can extend and modify the pipeline to fit your specific project requirements.
- Adjust the SonarQube analysis parameters and criteria as needed.

