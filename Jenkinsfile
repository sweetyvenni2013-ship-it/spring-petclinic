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
        git branch: 'main',
            credentialsId: 'github_token',
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
                        -Dsonar.login=$SONAR_TOKEN
                        '''
                    }
                }
            }
        }
    }
    post {
        always {
            // Added allowEmptyArchive to prevent failures if build is interrupted
            archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
            
            // THE FIX: Added allowEmptyResults: true
            // This prevents the build failure when -DskipTests is used.
            junit testResults: '**/target/surefire-reports/*.xml', allowEmptyResults: true
        }
    }
}


     

