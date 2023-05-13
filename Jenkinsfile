pipeline{
    agent any
    environment{
    VERSION="${env.BUILD_ID}"
    }
    stages{
        stage("Sonar Quality Checck"){
            // agent{
            //     docker {
            //         image 'openjdk:11'
            //     }
            // }
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                      sh 'chmod 777 gradlew'
                      sh './gradlew --status'
                      sh './gradlew sonarqube --warning-mode all'
                    }
                    // timeout(time: 1, unit: 'HOURS') {
                    //   def qg = waitForQualityGate()
                    //   if (qg.status != 'OK') {
                    //        error "Pipeline aborted due to quality gate failure: ${qg.status}"
                    //   }
                    // }
                }
            }
        }
        stage("docker build & docker push"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus_pass', variable: 'nexus_pass')]) {
                    sh '''
                    docker build -t 44.197.240.117:8083/springapp:${VERSION} .
                    docker login -u admin -p $nexus_pass 44.197.240.117:8083
                    docker push 44.197.240.117:8083/springapp:${VERSION}
                    docker rmi 44.197.240.117:8083/springapp:${VERSION}
                    '''
                }
            }
        }
        }
    }


}