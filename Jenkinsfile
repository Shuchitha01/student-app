pipeline {
    agent any

    environment {
        JFROG_URL  = "http://65.1.146.55:8082/artifactory"
        JFROG_REPO = "libs-snapshot-local"

        AWS_REGION = "ap-south-1"
        ECR_URI    = "581807542961.dkr.ecr.ap-south-1.amazonaws.com/student-app"
    }

    stages {

        stage('Checkout') {
            steps {
                  git branch: 'main',
                git 'https://github.com/Shuchitha01/student-app.git'
            }
        }

        stage('Build WAR') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Deploy WAR to JFrog') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'jfrog-creds',
                    usernameVariable: 'JF_USER',
                    passwordVariable: 'JF_PASS'
                )]) {
                    sh '''
                    mvn deploy -DskipTests \
                    -DaltDeploymentRepository=libs-snapshot-local::default::http://65.1.146.55:8082/artifactory/libs-snapshot-local \
                    -Dusername=$JF_USER \
                    -Dpassword=$JF_PASS
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t student-app:latest .'
            }
        }

        stage('Login to AWS ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                                  credentialsId: 'aws-ecr-creds']]) {
                    sh '''
                    aws ecr get-login-password --region ap-south-1 \
                    | docker login --username AWS --password-stdin 581807542961.dkr.ecr.ap-south-1.amazonaws.com
                    '''
                }
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                docker tag student-app:latest $ECR_URI:latest
                docker push $ECR_URI:latest
                '''
            }
        }

        stage('Pull Image from ECR') {
            steps {
                sh '''
                docker pull $ECR_URI:latest
                '''
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                docker rm -f student-app || true
                docker run -d \
                  --name student-app \
                  -p 8083:8080 \
                  $ECR_URI:latest
                '''
            }
        }
    }

    post {
        success {
            echo "✅ CI/CD completed successfully: App is running"
        }
        failure {
            echo "❌ CI/CD failed – check logs"
        }
    }
}
