pipeline{
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("Sonar Quality check"){
            agent{
                docker{
                    image 'openjdk:11'
                }
            }
            steps{
                script{
                    withSonarQubeEnv('sonarserver') {   
                        sh 'chmod +x gradlew'
                        sh './gradlew'           
                    }
                    timeout(time: 1, unit: 'HOURS') {
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
                        docker build -t 18.60.89.130:8083/springapp:${VERSION} .
                        docker login -u admin -p $docker_password 18.60.89.130:8083 
                        docker push  18.60.89.130:8083/springapp:${VERSION}
                        docker rmi 18.60.89.130:8083/springapp:${VERSION}                       
                        '''                                             
                    }
                }
            }
        }
    }
}
