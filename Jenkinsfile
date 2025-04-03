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
        stage('Trigger Hadoop Job') {
            when {
                expression {
                    return true
                }
            }
            steps {
                withEnv(['GCLOUD_PATH=/var/jenkins_home/google-cloud-sdk/bin']) {
                    withCredentials([file(credentialsId: 'google_auth', variable: 'GOOGLE_KEY')]) {
                        sh '''
                            $GCLOUD_PATH/gcloud auth activate-service-account --key-file=$GOOGLE_KEY
                            $GCLOUD_PATH/gcloud config set project ${PROJECT_ID}
                            $GCLOUD_PATH/gcloud config set compute/region ${REGION}
                            
                            // Upload mapper.py and reducer.py to GCS
                            gsutil cp mapper.py gs://${BUCKET}/mapper.py
                            gsutil cp reducer.py gs://${BUCKET}/reducer.py

                            // Submit the Hadoop job
                            $GCLOUD_PATH/gcloud dataproc jobs submit hadoop \
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
}
