pipeline {
    agent any
    options {
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '30', numToKeepStr: '2')
    }

    tools {
        maven 'Maven_Home'
    }

    environment {
        SCANNER_HOME = tool 'Sonar-scanner'
    }

    stages {
        stage('Git-Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/promo286/addressbook.git']])
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Sonar Scan') {
            steps {
                withSonarQubeEnv('sonarqube-scanner') {
                    sh """
                    "${SCANNER_HOME}/bin/sonar-scanner" \
                    -Dsonar.projectName=tamil-cloud-devops_buggy-app \
                    -Dsonar.projectKey=tamil-cloud-devops_buggy-app \
                    -Dsonar.organization=tamil-cloud-devops \
                    -Dsonar.java.binaries=target/classes
                    """
                }
            }
        }

        stage('Quality Gate Check') {
            steps {
                script {
                    def qualityGate = waitForQualityGate()

                    if (qualityGate.status == 'OK') {
                        echo 'Quality gate passed! Proceeding with the pipeline.'
                    } else {
                        error "Quality gate failed: ${qualityGate.status}. Stopping the pipeline."
                    }
                }
            }
        }

        stage('Snyk Scan') {
            steps {
                snykSecurity(
                    snykInstallation: 'Snyk_tool',
                    snykTokenId: 'snyk-api',
                    failOnError: false,
                    failOnIssues: false,
                    monitorProjectOnBuild: true
                )
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', 
                               odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }

    }
}
