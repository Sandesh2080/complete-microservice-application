pipeline {
    agent { label 'podman' }

    tools {
        maven 'M3'
    }

    environment {
        AWS_ACCOUNT_ID   = "${env.AWS_ACCOUNT_ID}"    // configure in job or via parameters
        AWS_REGION       = "${env.AWS_REGION ?: 'us-east-1'}"
        ECR_REPO_PREFIX  = "${env.ECR_REPO_PREFIX ?: 'myorg'}"

        // A comma‑separated list of module names matching subdirs
        SERVICES         = "api-gateway,offer-service,product-service,service-registry"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build JARs') {
            steps {
                sh 'mvn -f api-gateway/pom.xml clean install'
                sh 'mvn -f offer-service/pom.xml clean install'
                sh 'mvn -f product-service/pom.xml clean install'
                sh 'mvn -f service-registry/pom.xml clean install'
            }
        }

        stage('Build Docker images') {
            steps {
                script {
                    def mods = SERVICES.split(',')
                    mods.each { mod ->
                        sh """
                            podman build -t ${ECR_REPO_PREFIX}/${mod}:latest \\
                                      -f ${mod}/Dockerfile ${mod}
                        """
                    }
                }
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'AWS_CREDENTIALS'
                ]]) {
                    sh '''
                        aws --region ${AWS_REGION} ecr get-login-password \
                            | docker login \
                                --username AWS \
                                --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    '''
                }
            }
        }

        stage('Push images') {
            steps {
                script {
                    def mods = SERVICES.split(',')
                    mods.each { mod ->
                        sh """
                            docker tag ${ECR_REPO_PREFIX}/${mod}:latest \
                                       ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_PREFIX}/${mod}:latest

                            docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_PREFIX}/${mod}:latest
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo "Images pushed to ECR: ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_PREFIX}"
        }
    }
}
