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
                        withEnv(['DATREE_TOKEN=0963dd00-0171-4d1e-9804-b887cb4ff609']) {
                            sh 'helm datree test myapp/'
                        }
                    }
                }
            }
        }
        stage("Pushing helm charts to nexus"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                        dir('Kubernetes/') {
                            sh '''
                                helmversion=$( helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                                tar -czvf myapp-${helmversion}.tgz myapp/
                                curl -u admin:$docker_password http://34.125.168.1:8081/repository/helm-hosted/ --upload-file myapp-${helmversion}.tgz -v
                            '''
                        }
                    }
                }
            }
        }
        stage('Deploying app on k8s cluster') {
            steps {
                script{
                    withCredentials([kubeconfigContent(credentialsId: 'kubernetes-config', variable: 'KUBECONFIG_CONTENT')]) {
                        dir('Kubernetes/') {
                            sh 'helm upgrade --install --set image.repository="34.125.168.1:8083/springapp" --set image.tag="${VERSION}" myjavaapp myapp/'
                        }
                    }
                }
            }
        }
    }
}