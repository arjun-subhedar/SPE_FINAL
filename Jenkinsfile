pipeline {
    environment {
        DOCKERHUB_CRED = credentials("docker_credentials")
        // MONGO_URI = credentials("mongo-uri")
        // SECRET_KEY = credentials("cloud_secret_key")
        // CLOUD_NAME = credentials("cloud_name")
        // API_KEY = credentials("cloud_api_key")
        // API_SECRET = credentials("cloud_api_secret")
        PORT = "8000" 
        // MINIKUBE_HOME = '/home/jenkins/.minikube'
        // VAULT_PASS = credentials("ansible_vault_pass")
    }
    agent any
    tools {nodejs "NODEJS"} 
    stages {
        stage("Stage 1: Git Clone") {
            steps {
                sh '''
                [ -d SPE_FINAL ] && rm -rf SPE_FINAL
                git clone https://github.com/arjun-subhedar/SPE_FINAL
                '''
            }
        }

        // stage("Stage 2: Backend Testing") {
        //     steps {
        //         sh '''
        //         cd server
        //         npm i
        //         cd test
        //         npm install mocha chai sinon prom-client
        //         npm test
        //         '''
        //     }
        // }

        stage("Stage 3: Build frontend") {
            steps {
                sh '''
                cd SPE_FINAL/client
                npm install
                npm run build
                '''
            }
        }
        stage("Stage 3.5: Remove docker images and container") {
            steps {
                sh "docker container prune -f"
                sh "docker image prune -a -f"
            }
        }

        stage("Stage 4.1: Creating Docker Image for frontend") {
            steps {
                sh '''
                cd SPE_FINAL/client
                docker build -t pranav243/spe_main_project_client:latest .
                '''
            }
        }
        stage("Stage 4.2: Scan Docker Image for frontend") {
            steps {
                sh '''
                trivy image pranav243/spe_main_project_client:latest
                '''
            }
        }
        stage("Stage 4.3: Push Frontend Docker Image") {
            steps {
                sh '''
                docker login -u ${DOCKERHUB_CRED_USR} -p ${DOCKERHUB_CRED_PSW}
                docker push pranav243/spe_main_project_client:latest
                '''
            }
        }
        

        stage("Stage 5.1: Creating Docker Image for backend") {
            steps {
                sh '''
                cd SPE_FINAL/server
                docker build -t pranav243/spe_main_project_server:latest .
                '''
            }
        }
        stage("Stage 5.2: Scan Docker Image for backend") {
            steps {
                sh '''
                trivy image pranav243/spe_main_project_server:latest
                '''
            }
        }


        stage("Stage 5.3: Push Backend Docker Image") {
            steps {
                sh '''
                docker login -u ${DOCKERHUB_CRED_USR} -p ${DOCKERHUB_CRED_PSW}
                docker push pranav243/spe_main_project_server:latest
                '''
            }
        }

        stage("Stage 8: Ansible"){
            steps {
                sh '''
                cd SPE_FINAL
                ansible-playbook -i inventory-k8 playbook-k8.yaml
                '''
            }
        }
    }
}