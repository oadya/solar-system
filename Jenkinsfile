pipeline {

    agent any

    environment {
      NAME = "solar-system"
      VERSION = "${env.BUILD_ID}-${env.GIT_COMMIT}"
      IMAGE_REPO = "orpheadya"
      //ARGO_TOKEN = credentials('argo-token')
      GITHUB_TOKEN = credentials('github-token')
      NEW_IMAGE_NAME = ''
      GIT_REPO_NAME = "gitops-argocd"
      GIT_USER_NAME = "oadya"
    }
    
    
    stages {

        stage('Unit Tests') {
            steps {
                echo 'Implement unit tests if applicable'
                echo 'This stage is a sample placeholder'
            }
        }
        
        stage('Build Image') {
            steps {
              sh "docker build -t ${NAME} ."
              sh "docker tag ${NAME}:latest ${IMAGE_REPO}/${NAME}:${VERSION}"
            }
     
        }

       // Push image to docker registry
        stage("Push Image") {
            steps {
               withDockerRegistry([credentialsId: 'docker', url: ""]) {

                    sh 'docker push ${IMAGE_REPO}/${NAME}:${VERSION}'
                        
                }
            }
        }

       // Update kubernetes manifest deployments  with the new version of the image
        stage("Clone or Pull Manifest Repo") {
            steps {

               script {

                  if(fileExists('gitops-argocd')) {

                    echo 'Cloned repo already exists - Pulling the latest changes'

                    dir('gitops-argocd') {
                        sh 'git pull'
                    }

                  } else {

                    echo 'Repo does not exists - Cloning the repo'
                    sh 'git clone -b feature-gitea https://github.com/oadya/gitops-argocd'

                  }
               }
            }
        }

        stage('Update manifest') {
            steps {
                script {
                    // Determine the image name dynamically based on your versioning strategy
                    NEW_IMAGE_NAME = "${IMAGE_REPO}/${NAME}:${VERSION}"
                    dir('gitops-argocd/jenkins-demo') {
                        sh "sed -i 's|image: .*|image: $NEW_IMAGE_NAME|' deployment.yml"
                        sh 'cat tdeployment.yml'
                    }
              }
              
            }
     
        }

        stage('Commit & Push') {
            steps {
                dir('gitops-argocd/jenkins-demo') {
                    sh "git remote set-url origin https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}"
                    sh 'git checkout feature-gitea'
                    sh 'git add -A'
                    sh 'git commit -am "Updated image version for build - $VERSION"'
                    sh 'git push origin feature-gitea'
                }
            }
     
        }

        /*
        stage('Raise a Pull Request') {
            steps {
              sh "bash pr.sh"
            }
     
        }
        */
    
       
    }
    
    
}


