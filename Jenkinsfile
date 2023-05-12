pipeline{
    agent any
    stages{
        stage("Sonar Quality Checck"){
            agent{
                docker {
                    image 'openjdk:11'
                }
            }
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                      sh 'chmod +x gradlew'
                      sh './gradlew --status'
                      sh './gradlew --daemon'
                      sh './gradlew --status'
                      sh './gradlew sonarqube --debug'
                    }
                    
                }
            }
        }
    }
}