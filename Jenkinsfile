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
                // Using 'github_token' as verified in your credentials screenshot
                git branch: 'main',
                    credentialsId: 'github_token', 
                    url: 'https://github.com'
            }
        }

        stage('Build & Sonar Scan') {
            steps {
                withCredentials([string(credentialsId: 'sonar_id', variable: 'SONAR_TOKEN')]) {
                    withSonarQubeEnv('SONAR') {
                        // REMOVED: -DskipTests to allow test execution and report generation
                        sh '''
                        mvn clean verify sonar:sonar \
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
            // Archives the JAR file created in the target folder
            archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
            
            // Collects the JUnit XML reports generated because we REMOVED -DskipTests
            junit testResults: '**/target/surefire-reports/*.xml', allowEmptyResults: true
        }
    }
}

  
  


     

