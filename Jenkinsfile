
pipeline {
    
    agent any 
    
    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        
        stage('Checkout'){
           steps {
                script {
                    checkout scmGit(
                        branches: [[name: '*/main']],
                        extensions: [],
                        userRemoteConfigs: [[credentialsId: '62b90739-5092-4897-adac-f2f831f5d496', url: 'https://github.com/vishalmanchanda143/python-jenkin-project']]
                    )
                }
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
            environment {
            GIT_REPO_NAME = "cicd-demo-manifests-repo"
            GIT_USER_NAME = "vishalmanchanda143"
        }
            steps {
                script{
                     withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                        sh '''
                        cat deploy.yaml
                        sed -i "s/image:/image:${BUILD_NUMBER}/g" deploy.yaml
                        cat deploy.yaml
                        git add deploy.yaml
                        git commit -m 'Updated the deploy yaml | Jenkins Pipeline'
                        git remote -v
                        git config --list
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                        '''                        
                    }
                }
            }
        }
    }
post {
        success {
            updateGitHubCommitStatus('success', 'Build succeeded!', "${env.BUILD_URL}")
        }
        failure {
            updateGitHubCommitStatus('failure', 'Build failed!', "${env.BUILD_URL}")
        }
    }
}

def updateGitHubCommitStatus(String state, String description) {
    withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
        def commitStatus = [
            state: state,
            context: 'jenkins/build',
            description: description,
            target_url: "${env.BUILD_URL}"
        ]

        def repoUrl = scm.getUserRemoteConfigs()[0].getUrl()
        def repoOwner = repoUrl.split('/')[3]
        def repoName = repoUrl.split('/')[4].replace('.git', '')
        def apiUrl = "https://api.github.com/repos/${repoOwner}/${repoName}/statuses/${env.GIT_COMMIT}"

        sh """
            curl -s -H "Authorization: token ${GITHUB_TOKEN}" \
            -H "Content-Type: application/json" \
            -d '${new groovy.json.JsonBuilder(commitStatus).toString()}' \
            ${apiUrl}
        """
    }

}
