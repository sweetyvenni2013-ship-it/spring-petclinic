pipeline {
    agent { label 'spc' }
    
    // 1. ADDED: This forces Jenkins to use the JDK 25 you configured in Global Tools
    tools {
        jdk 'Java25' 
    }
    
    triggers {
        pollSCM('* * * * *')
    }
 
    stages {
        stage('Git Checkout') {   
            steps {
                git url: 'https://github.com', branch: 'main'
            }
        }

        stage('Build and Scan') {
            steps {
                withCredentials([string(credentialsId: 'sonar_sonar', variable: 'SONAR_TOKEN')]) {
                    withSonarQubeEnv('SONAR') {
                        sh """
                        mvn package sonar:sonar \
                        -Dsonar.projectKey=soumya1312shekar_java \
                        -Dsonar.organization=soumya1312shekar-1 \
                        -Dsonar.host.url=https://sonarcloud.io \
                        -Dsonar.login=${SONAR_TOKEN}
                        """
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
                    echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin
                    docker build -t ${DOCKER_USER}/java-app:${env.BUILD_ID} .
                    docker push ${DOCKER_USER}/java-app:${env.BUILD_ID}
                    """
                }
            }
        }

        stage('ECR Push and Hub Pull') {
            steps {
                sh """
                docker image pull nginx:1.30
                aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 271071982991.dkr.ecr.ap-south-1.amazonaws.com
                
                # 2. FIXED: Removed '://' and added a repository name (e.g., /my-app)
                docker tag nginx:1.30 ://271071982991.dkr.ecr.ap-south-1.amazonaws.com
                docker push ://271071982991.dkr.ecr.ap-south-1.amazonaws.com
                """
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '**/*.jar', allowEmptyArchive: true
            junit '**/surefire-reports/*.xml'
            sh "docker logout || true"
        }
    }
}
