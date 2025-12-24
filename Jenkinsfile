pipeline {
    agent any

    environment {
        // JFrog
        JFROG_URL  = "http://13.200.200.175:8082/artifactory"
        JFROG_REPO = "libs-snapshot-local"

        // AWS / ECR
        AWS_REGION = "ap-south-1"
        AWS_ACCOUNT_ID = "581807542961"
        ECR_REPO = "student-app"
        IMAGE_TAG = "latest"
        ECR_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"
    }

    stages {

        /* -------------------- 1. CHECKOUT -------------------- */
        stage('Checkout Code') {
            steps {
                git credentialsId: 'github-creds',
                    url: 'https://github.com/Shuchitha01/student-app.git',
                    branch: 'main'
            }
        }

        /* -------------------- 2. BUILD WAR -------------------- */
        stage('Build WAR') {
            steps {
                sh 'mvn clean package'
            }
        }

        /* -------------------- 3. DEPLOY WAR TO JFROG -------------------- */
        stage('Deploy WAR to JFrog') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'jfrog-creds',
                    usernameVariable: 'JF_USER',
                    passwordVariable: 'JF_PASS'
                )]) {
                    sh """
                    mvn deploy -DskipTests \
                    -DaltDeploymentRepository=${JFROG_REPO}::default::${JFROG_URL}/${JFROG_REPO} \
                    -Dusername=${JF_USER} \
                    -Dpassword=${JF_PASS}
                    """
                }
            }
        }

        /* -------------------- 4. BUILD DOCKER IMAGE -------------------- */
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${ECR_REPO}:${IMAGE_TAG} .'
            }
        }

        /* -------------------- 5. LOGIN TO AWS ECR -------------------- */
  stage('Login to AWS ECR') {
    steps {
        withCredentials([[
            $class: 'AmazonWebServicesCredentialsBinding',
            credentialsId: 'aws-ecr-creds'
        ]]) {
            sh '''
              aws sts get-caller-identity
              aws ecr get-login-password --region ap-south-1 \
              | docker login --username AWS --password-stdin 581807542961.dkr.ecr.ap-south-1.amazonaws.com
            '''
        }
    }
}


        /* -------------------- 6. PUSH IMAGE TO ECR -------------------- */
        stage('Push Image to ECR') {
            steps {
                sh '''
                docker tag ${ECR_REPO}:${IMAGE_TAG} ${ECR_URI}:${IMAGE_TAG}
                docker push ${ECR_URI}:${IMAGE_TAG}
                '''
            }
        }

        /* -------------------- 7. RUN CONTAINER -------------------- */
        stage('Run Container') {
            steps {
                sh '''
                docker rm -f student-app || true
                docker run -d \
                  --name student-app \
                  -p 8083:8080 \
                  ${ECR_URI}:${IMAGE_TAG}
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Checkout → WAR → JFrog → Docker → ECR → Container completed successfully"
        }
        failure {
            echo "❌ Pipeline failed. Check logs."
        }
    }
}
