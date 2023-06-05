
pipeline{
    agent any 
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("sonar quality check"){
            agent {
                docker {
                    image 'openjdk:11'
                }
            }
            steps{
                script{withSonarQubeEnv(credentialsId: 'sonar_token') {
                               sh 'chmod +x gradlew'
                               sh './gradlew sonarqube'
                    }

                    timeout(time: 5, unit: 'MINUTES') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }

                }  
            }
        }
        stage("docker build & docker push"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                             sh '''
                                docker build -t 3.110.219.65:8083/springapp:${VERSION} .
                                docker login -u admin -p $docker_password 3.110.219.65:8083 
                                docker push  3.110.219.65:8083/springapp:${VERSION}
                                docker rmi 3.110.219.65:8083/springapp:${VERSION}
                            '''
                    }
                }
            }
        }
    }
}