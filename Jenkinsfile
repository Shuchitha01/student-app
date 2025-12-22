pipeline {
    agent any

    environment {
        JFROG_URL  = "http://65.1.146.55:8082/artifactory"
        JFROG_REPO = "libs-snapshot-local"
    }

    stages {

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
                    sh """
                    mvn deploy -DskipTests \
                    -DaltDeploymentRepository=${JFROG_REPO}::default::${JFROG_URL}/${JFROG_REPO} \
                    -Dusername=$JF_USER \
                    -Dpassword=$JF_PASS
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t student-app:latest .'
            }
        }
    }

    post {
        success {
            echo "✅ Git → WAR → JFrog → Docker completed successfully"
        }
    }
}
