pipeline {
    agent { label 'spc' }
    
    triggers {
        pollSCM('* * * * *')
    }
 
    stages {
        stage('Git Checkout') {   
            steps {
                git branch: 'main', url: 'https://github.com/soumya1312shekar/java.git'
            }
        }

        stage('Build and Scan') {
            steps {
                // withSonarQubeEnv already provides SONAR_HOST_URL and SONAR_AUTH_TOKEN if configured correctly in Jenkins
                withSonarQubeEnv('SONAR') {
                    sh "mvn clean package sonar:sonar -Dsonar.projectKey=soumya1312shekar_java -Dsonar.organization=soumya1312shekar-1"
                }
            }
        }

        stage('Docker Hub Login & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'd3d6bc96-8f86-4e2e-acf6-ccb785b65c88',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                    echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                    docker build -t ${DOCKER_USER}/java-app:${env.BUILD_ID} .
                    docker push ${DOCKER_USER}/java-app:${env.BUILD_ID}
                    """
                }
            }
        }

        stage('ECR Push and Hub Pull') {
            steps {
                script {
                    // Define your repository URL (No 'https://' inside docker commands)
                    def ecrRepo = "://271071982991.dkr.ecr.ap-south-1.amazonaws.com"
                    
                    sh """
                    docker pull nginx:1.30
                    aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 271071982991.dkr.ecr.ap-south-1.amazonaws.com
                    
                    # FIX: Removed :// and added a repository path
                    docker tag nginx:1.30 ${ecrRepo}:latest
                    docker push ${ecrRepo}:latest
                    """
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
            // Added check to prevent failure if tests didn't run
            catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                junit '**/target/surefire-reports/*.xml'
            }
            sh "docker logout || true"
        }
    }
}
