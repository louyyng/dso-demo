// Jenkinsfile
pipeline {
    // Agent definition now points to our updated build-agent.yaml
    agent {
        kubernetes {
            yamlFile 'build-agent.yaml'
            defaultContainer 'maven'
        }
    }

    // Environment variables available to all stages
    environment {
        // !!! IMPORTANT !!!
        // Replace the value below with the ECR Repository URI you copied earlier.
        ECR_REPO_URI = 'ACCOUNT_ID.dkr.ecr.us-east-2.amazonaws.com/dso-demo'
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

        // This stage now includes both packaging and publishing the image
        stage('Package & Publish') {
            parallel {
                stage('Create Jarfile') {
                    steps {
                        sh 'mvn package -DskipTests'
                    }
                }

                // This is the NEW stage we are adding!
                stage('Build and Push Image to ECR') {
                    steps {
                        // We switch to the 'kaniko' container defined in our build-agent.yaml
                        container('kaniko') {
                            // Automatically get the first 8 characters of the git commit hash for the image tag
                            def imageTag = env.GIT_COMMIT.take(8)
                            
                            // The Kaniko command to build and push the image
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
