pipeline{
    agent any
    stages{
        stage("Sonar Quality Check"){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                            sh 'chmod +x gradlew'
                            sh './gradlew sonarqube'
                    }
                }
            }
        }
        stage("Quality Gate Status"){
            steps{
                    waitForQualityGate abortPipeline: true
        }
    }
    
}