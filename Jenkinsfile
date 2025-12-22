pipeline {
    agent any

    environment {
        JFROG_URL  = "http://65.1.146.55:8082/artifactory"
        JFROG_REPO = "libs-snapshot-local"
    }

    stages {

        stage('Git Clone') {
            steps {
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
                sh """
                mvn deploy \
                -DaltDeploymentRepository=jfrog::default::${JFROG_URL}/${JFROG_REPO}
                """
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
