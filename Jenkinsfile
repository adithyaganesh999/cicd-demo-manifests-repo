pipeline {
    agent any
    
    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        
        stage('Build Docker') {
            steps {
                script {
                    sh '''
                    echo 'Building Docker Image'
                    docker build -t adithyaganesh640/todo-app:${BUILD_NUMBER} .
                    '''
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                        sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push adithyaganesh640/todo-app:${BUILD_NUMBER}
                        '''
                    }
                }
            }
        }
        
        stage('Update K8S Manifest') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github-creds', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        sh '''
                        rm -rf cicd-demo-manifests-repo
                        git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/adithyaganesh999/cicd-demo-manifests-repo.git
                        cd cicd-demo-manifests-repo
                        sed -i "s|image:.*|image: adithyaganesh640/todo-app:${BUILD_NUMBER}|g" deploy/deploy.yaml
                        git config user.email "jenkins@pipeline.com"
                        git config user.name "Jenkins"
                        git add deploy/deploy.yaml
                        git commit -m "Updated image to build ${BUILD_NUMBER}"
                        git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/adithyaganesh999/cicd-demo-manifests-repo.git HEAD:main
                        '''
                    }
                }
            }
        }
    }
}
