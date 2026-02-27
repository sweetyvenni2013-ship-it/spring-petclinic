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
                // FIX 1: Changed 'github_token' to 'my_jenk_id' (from your logs)
                git branch: 'main',
                    credentialsId: 'my_jenk_id', 
                    url: 'https://github.com/soumya1312shekar/java.git'
            }
        }

        stage('Build & Sonar Scan') {
            steps {
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
            // FIX 2: Added proper spacing and narrow path for artifacts
            archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
            // FIX 3: Added 'target' to the path to match Maven structure
            junit testResults: '**/target/surefire-reports/*.xml', allowEmptyResults: true
        }
    }
}

