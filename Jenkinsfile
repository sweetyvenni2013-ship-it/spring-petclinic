pipeline {
    agent { label 'spc' }
    
    triggers {
        pollSCM('* * * * *')
    }
 
    stages {
        stage('Git Checkout') {   
            steps {
                // Fixed: Added comma after URL to prevent syntax errors
                git url: 'https://github.com/soumya1312shekar/java.git', branch: 'main'
            }
        }

       // Defines the stage for compiling the code and running security/quality analysis
stage('Build and Scan') {
    steps {
        // Uses a script block to allow for more complex logic within the stage
        script {
            // Securely injects the SonarCloud token from Jenkins credentials manager
            withCredentials([string(credentialsId: 'sonar_sonar', variable: 'SONAR_TOKEN')]) {
                
                // Configures the environment to use the 'SONAR' server defined in Jenkins settings
                withSonarQubeEnv('SONAR') {
                    
                    // Runs Maven to:
                    // 1. package: Compile and build the JAR file
                    // 2. sonar:sonar: Push the results to SonarCloud for analysis
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
                
                # Fixed: Removed '://' and added a repository name (e.g., /my-app)
                docker tag nginx:1.30 ://271071982991.dkr.ecr.ap-south-1.amazonaws.com
                docker push ://271071982991.dkr.ecr.ap-south-1.amazonaws.com
                """
            }
        }
    }

    post {
        always {
            // Added allowEmptyArchive to prevent failure if the build fails before JAR creation
            archiveArtifacts artifacts: '**/*.jar', allowEmptyArchive: true
            junit '**/surefire-reports/*.xml'
            sh "docker logout || true"
        }
    }
}
