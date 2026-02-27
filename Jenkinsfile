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
                // Using 'my_jenk_id' as verified in your Jenkins logs
                git branch: 'main',
                    credentialsId: 'my_jenk_id', 
                    url: 'https://github.com'
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
            // Archives the JAR file created in the target folder
            archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
            // Records the test results from the surefire reports
            junit testResults: '**/target/surefire-reports/*.xml', allowEmptyResults: true
        }
    }
}


  

     

