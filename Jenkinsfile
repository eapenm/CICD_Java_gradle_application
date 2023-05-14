pipeline{
    agent any
    environment{
    VERSION="${env.BUILD_ID}"
    KUBECONFIG = "${WORKSPACE}/kubeconfig/kubeconfig.yaml"
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
                    docker build -t 3.239.228.172:8083/springapp:${VERSION} .
                    docker login -u admin -p $nexus_pass 3.239.228.172:8083
                    docker push 3.239.228.172:8083/springapp:${VERSION}
                    docker rmi 3.239.228.172:8083/springapp:${VERSION}
                    '''
                }
            }
        }
        }
        stage("identifying misconfigs using datree in helm charts"){
            steps{
                script{
                       dir('kubernetes/') {
                    //         withEnv(['DATREE_TOKEN=960ba681-237c-4a8c-abcb-5744fdd516c6']) {
                    //           sh 'helm datree test myapp/'
                    //         }
                     echo "datree policy check"
                     }
                }
            }

        }
        stage("Push the helm chart to Nexus"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus_pass', variable: 'nexus_pass')]) {
                        dir('kubernetes/') {
                            sh '''
                            helmversion=$(helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                            tar -czvf myapp-${helmversion}.tgz myapp/
                            curl -u admin:$nexus_pass http://3.239.228.172:8081/repository/helm-hosted-eap/ --upload-file myapp-${helmversion}.tgz -v

                            '''
                        }
                }
            }
        }
        }
        stage('Deploying application to K8 Cluster') {
            steps {
                script{
                   withCredentials([kubeconfigFile(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                         dir('kubernetes/') {
                          sh 'helm upgrade --install --set image.repository="3.239.228.172:8083/springapp" --set image.tag="${VERSION}" myjavaapp myapp/ ' 
                        }
                }
                }
            }
        }
    }
post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "eapenmani@gmail.com";  
		}
	}

}