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
                      sh 'chmod 777 gradlew'
                      sh './gradlew --status'
                      sh './gradlew sonarqube --debug'
                    }
                    
                }
            }
        }
    }
}