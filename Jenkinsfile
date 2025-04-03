pipeline {
    agent any
    environment {
        PROJECT_ID = 'cmu-14848-449300'
        REGION = 'us-west1'
        BUCKET = 'project_bucket_14848_hadoop'
        CLUSTER = 'project-hadoop'
        SERVICE_ACCOUNT_KEY = '/path/to/service-account.json'
        scannerHome = tool 'sonarqube'
    }
    
    stages {
        stage('Fetch Code') {
            steps {
                git branch: 'main', 
                url: 'https://github.com/number000000/14848-project-dummy-repo.git'
            }
        }
        stage('Code Analysis') {
            steps {
                script {
                    withSonarQubeEnv('sonarqube') {
                        sh "${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=14848project \
                            -Dsonar.projectName=14848project \
                            -Dsonar.sources=."
                    }
                }
            }
        }
        stage('Quality Gate Check') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    timeout(time: 5, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }
    }
}
