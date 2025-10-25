// Jenkinsfile (FINAL CORRECTED VERSION)
pipeline {
    // We added the serviceAccount line here to fix the ECR permission denied error
    agent {
        kubernetes {
            yamlFile 'build-agent.yaml'
            defaultContainer 'maven'
            serviceAccount 'jenkins'
        }
    }

    environment {

        ECR_REPO_URI = '751910243184.dkr.ecr.us-east-2.amazonaws.com/dso-demo' 
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
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
