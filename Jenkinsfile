pipeline{
    agent any

    environment {
        VENV_DIR = 'venv'
        GCP_PROJECT = "atomic-horizon-464016-q9"
        GCLOUD_PATH = "/var/jenkins-home/google-cloud-sdk/bin"
    }

    stages{
        stage('Cloning github repo to Jenkins'){
            steps{
                script{
                    echo 'Cloning github repo to Jenkins.......'
                    checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'github-token-1', url: 'https://github.com/nandaryanpk/mlops-project-1.git']])
                }
            }
        }

        stage('Setting up our Virtual Environment and Installing dependencies'){
            steps{
                script{
                    echo 'Setting up our Virtual Environment and Installing dependencies.......'
                    sh '''
                    python -m venv ${VENV_DIR}
                    . ${VENV_DIR}/bin/activate
                    pip install --upgrade pip
                    pip install -e .
                    '''                   
                }
            }
        }

        stage('Building and Pushing Docker Image to GCR'){
            steps{
                withCredentials([file(credentialsId : 'gcp-key-mlops1', variable : 'GOOGLE_APP_CREDENTIALS')]){
                    script{
                        echo 'Building and Pushing Docker Image to GCR.......'
                        sh'''
                        export PATH=$PATH:${GCLOUD_PATH}

                        gcloud auth activate-service-account --key-file=${GOOGLE_APP_CREDENTIALS}

                        gcloud config set project ${GCP_PROJECT}

                        gcloud auth configure-docker --quiet

                        docker build -t gcr.io/${GCP_PROJECT}/ml-project:latest .

                        docker push gcr.io/${GCP_PROJECT}/ml-project:latest
                        '''
                    }
                }
            }
        }

        stage('Deploy to Google Cloud Run'){
            steps{
                withCredentials([file(credentialsId : 'gcp-key-mlops1', variable : 'GOOGLE_APP_CREDENTIALS')]){
                    script{
                        echo 'Deploy to Google Cloud Run.......'
                        sh'''
                        export PATH=$PATH:${GCLOUD_PATH}

                        gcloud auth activate-service-account --key-file=${GOOGLE_APP_CREDENTIALS}

                        gcloud config set project ${GCP_PROJECT}

                        gcloud run deploy ml-project \
                            --image=gcr.io/${GCP_PROJECT}/ml-project:latest \
                            --platform=managed \
                            --region=us-central1 \
                            --allow-unauthenticated \
                            --port=8080
                             
                        '''
                    }
                }
            }
        }
    }
}