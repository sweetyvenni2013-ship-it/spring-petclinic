pipeline {
    agent { label 'spc' }
    
    triggers {
        pollSCM('* * * * *')
    }
 
    stages {
        stage('Git Checkout') {   
            steps {
                git url: 'https://github.com/soumya1312shekar/java.git', branch: 'main'
            }
        }

        stage('Build and Scan') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'sonar_sonar', variable: 'SONAR_TOKEN')]) {
                        withSonarQubeEnv('SONAR') {
                            // Added -DskipTests if you want to speed up the scan, 
                            // or leave as is to include test results in SonarCloud
                            sh "mvn package sonar:sonar \
                                -Dsonar.projectKey=soumya1312shekar_java \
                                -Dsonar.organization=soumya1312shekar-1 \
                                -Dsonar.host.url=https://sonarcloud.io \
                                -Dsonar.login=${SONAR_TOKEN}"
                        }
                    }
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
                    docker build -t "${DOCKER_USER}/java-app:${env.BUILD_ID}" .
                    docker push "${DOCKER_USER}/java-app:${env.BUILD_ID}"
                    """
                }
            }
        }

        stage('ECR Push') {
            steps {
                sh """
                # Authenticate and Push to ECR (ensure repository 'java-repo' exists)
                aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 271071982991.dkr.ecr.ap-south-1.amazonaws.com
                docker image pull nginx:1.25
                docker tag nginx:1.25 ://271071982991.dkr.ecr.ap-south-1.amazonaws.com
                docker push ://271071982991.dkr.ecr.ap-south-1.amazonaws.com
                """
            }
        }
    }

    post {
        always {
            // Use precise target path for spring-petclinic
            archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
            junit '**/target/surefire-reports/*.xml'
            sh "docker logout || true"
        }
    }
}
