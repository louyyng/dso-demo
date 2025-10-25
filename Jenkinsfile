pipeline {
    agent {
        kubernetes {
            yamlFile 'build-agent.yaml'
            defaultContainer 'maven'
            serviceAccount 'jenkins'
        }
    }

    environment {
        // Make sure this URI is correct!
        ECR_REPO_URI = '751910243184.dkr.ecr.us-east-2.amazonaws.com/dso-demo'
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn compile'
            }
        }

        // The 'Test' stage is now renamed to 'Static Analysis' and expanded
        stage('Static Analysis') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'mvn test'
                    }
                }

                // New Stage: Scan for vulnerabilities in dependencies
                stage('SCA') {
                    steps {
                        container('maven') {
                            // catchError makes the pipeline continue even if vulnerabilities are found
                            catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                                sh 'mvn org.owasp:dependency-check-maven:check'
                            }
                        }
                    }
                    // This post-action archives the HTML report for viewing in Jenkins
                    post {
                        always {
                            archiveArtifacts allowEmptyArchive: true, artifacts: 'target/dependency-check-report.html', fingerprint: true
                        }
                    }
                }

                // New Stage: Check for software licenses
                stage('OSS License Checker') {
                    steps {
                        container('licensefinder') {
                            // This command installs and runs license_finder
                            sh 'license_finder --decisions-file=./.github/decisions.yml'
                        }
                    }
                }
            }
        }

        stage('Package & Publish') {
            parallel {
                stage('Create Jarfile') {
                    steps {
                        sh 'mvn package -DskipTests'
                    }
                }

                stage('Build and Push Image to ECR') {
                    steps {
                        container('kaniko') {
                            script {
                                def imageTag = env.GIT_COMMIT.take(8)
                                
                                sh """
                                  /kaniko/executor \
                                    --context `pwd` \
                                    --dockerfile `pwd`/Dockerfile \
                                    --destination ${ECR_REPO_URI}:${imageTag} \
                                    --destination ${ECR_REPO_URI}:latest \
                                    --cache=true
                                """
                            }
                        }
                    }
                }
            }
        }
    }
}
