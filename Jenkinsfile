pipeline{
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("Sonar Quality Check"){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                            sh 'chmod +x gradlew'
                            sh './gradlew sonarqube'
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
        stage("Docker build && Docker Push"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                            sh '''
                            docker build -t 34.125.168.1:8083/springapp:${VERSION} .
                            docker login -u admin -p $docker_password 34.125.168.1:8083
                            docker push 34.125.168.1:8083/springapp:${VERSION}
                            docker rmi 34.125.168.1:8083/springapp:${VERSION}
                            '''
                    }
                    
                }
            }

        }
        stage("Identifying misconfigs using datree in helm charts"){
            steps{
                script{
                    dir('Kubernetes/') {
                        withEnv(['DATREE_TOEKN=0963dd00-0171-4d1e-9804-b887cb4ff609']) {
                            sh 'helm datree test myapp/'
                        }
                            
                    }
                }
            }
        }
    }
}