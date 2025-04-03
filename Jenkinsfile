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
                            -Dsonar.projectVersion=1.0 \
                            -Dsonar.sources=."
                    }
                }
            }
        }
        stage('Quality Gate Check') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Trigger Hadoop Job') {
            when {
                expression {
                    return true
                }
            steps {
                withCredentials([file(credentialsId: 'google_auth', variable: 'GOOGLE_KEY')]) {
                    sh '''
                        gcloud auth activate-service-account --key-file=$GOOGLE_KEY
                        gcloud config set project ${PROJECT_ID}
                        gcloud config set compute/region ${REGION}
                        
                        // Upload mapper.py and reducer.py to GCS
                        gsutil cp mapper.py gs://${BUCKET}/mapper.py
                        gsutil cp reducer.py gs://${BUCKET}/reducer.py

                        // Submit the Hadoop job
                        gcloud dataproc jobs submit hadoop \
                            --cluster=${CLUSTER} \
                            --region=${REGION} \
                            --files=gs://${BUCKET}/mapper.py, gs://${BUCKET}/reducer.py \
                            -- -mapper "python mapper.py" -reducer "python reducer.py" -input /data/*/ -output /HadoopOutput
                    '''
                }
            }
        }
    }
}
