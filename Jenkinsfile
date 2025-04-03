pipeline {
    agent any
    
    stages {
        stage('Fetch Code') {
            steps {
                git branch: 'main', 
                url: 'https://github.com/number000000/14848-project-dummy-repo.git'
            }
        }
        stage('Code Analysis') {
            environment {
                scannerHome = tool 'sonarqube'
            }
            steps {
                script {
                    withSonarQubeEnv('sonarqube') {
                        sh "${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=14848project \
                            -Dsonar.projectName=14848project \
                            -Dsonar.projectVersion=1.0 \
                            -Dsonar.sources=."
                    }
                }
            }
        }
    }
}
