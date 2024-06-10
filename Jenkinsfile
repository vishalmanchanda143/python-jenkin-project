
pipeline {
    
    agent any 
    
    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        
        stage('Checkout'){
           steps {
                git credentialsId: '62b90739-5092-4897-adac-f2f831f5d496', 
                url: 'https://github.com/vishalmanchanda143/python-jenkin-project',
                branch: 'main'
           }
        }

        stage('Build Docker'){
            steps{
                script{
                    sh '''
                    echo 'Buid Docker Image'
                    docker build -t vishal143/cicd-e2e:${BUILD_NUMBER} .
                    '''
                }
            }
        }

        stage('Push the artifacts'){
           steps{
                script{
                    echo 'Build Docker Image'
                    withCredentials([usernamePassword(credentialsId: 'docker-cred', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}"
                        sh "docker push vishal143/cicd-e2e:${BUILD_NUMBER}"
                    }
                }
            }
        }
        
        stage('Checkout K8S manifest SCM'){
            steps {
                git credentialsId: '62b90739-5092-4897-adac-f2f831f5d496', 
                url: 'https://github.com/vishalmanchanda143/cicd-demo-manifests-repo.git',
                branch: 'main'
            }
        }
        
        stage('Update K8S manifest & push to Repo'){
            steps {
                script{
                     withCredentials([usernamePassword(credentialsId: '62b90739-5092-4897-adac-f2f831f5d496', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'vishalmanchanda143')]) {
                        sh '''
                        cat deploy.yaml
                        sed -i "s/image:/image:${BUILD_NUMBER}/g" deploy.yaml
                        cat deploy.yaml
                        git add deploy.yaml
                        git commit -m 'Updated the deploy yaml | Jenkins Pipeline'
                        git remote -v
                        git push origin HEAD:main
                        '''                        
                    }
                }
            }
        }
    }
}
