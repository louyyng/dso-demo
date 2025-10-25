// Jenkinsfile (Corrected Version)
pipeline {
    agent {
        kubernetes {
            yamlFile 'build-agent.yaml'
            defaultContainer 'maven'
        }
    }

    environment {
        // Make sure this URI is correct!
        ECR_REPO_URI = '123456789012.dkr.ecr.us-east-2.amazonaws.com/dso-demo' // <-- 記得換成你自己的 URI
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
                            // The script block fixes the "Expected a step" error
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
