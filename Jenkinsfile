pipeline {
    agent { label 'spc' }

    tools {
        jdk 'jdk-17'
        maven 'Maven3'
    }

    environment {
        MAVEN_OPTS = '-Xmx1024m'
    }

    stages {
        stage('Checkout') {
            steps {
                // This ID now matches your 'github_token' credential in the image
                git branch: 'main',
                    credentialsId: 'github_token', 
                    url: 'https://github.com'
            }
        }

        stage('Build & Sonar Scan') {
            steps {
                // This matches 'sonar_id' from your image
                withCredentials([string(credentialsId: 'sonar_id', variable: 'SONAR_TOKEN')]) {
                    withSonarQubeEnv('SONAR') {
                        sh '''
                        mvn clean verify sonar:sonar \
                        -DskipTests \
                        -Dsonar.projectKey=soumya1312shekar_java \
                        -Dsonar.organization=soumya1312shekar-1 \
                        -Dsonar.host.url=https://sonarcloud.io \
                        -Dsonar.token=$SONAR_TOKEN
                        '''
                    }
                }
            }
        }
    }
    
    post {
        always {
            archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
            junit testResults: '**/target/surefire-reports/*.xml', allowEmptyResults: true
        }
    }
}

  



     

