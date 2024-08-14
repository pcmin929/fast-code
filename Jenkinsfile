pipeline {
    agent any
    environment {
        GIT_NAME = 'pcmin929'
        GIT_EMAIL = 'pcmin929@gmail.com'
        // CODE_REPO = 'git@github.com:pcmin929/fast-code.git'
        // MANIFEST_REPO = 'git@github.com:pcmin929/deployment.git'

        CODE_REPO = 'git-codecommit.ap-northeast-2.amazonaws.com/v1/repos/code'
        MANIFEST_REPO = 'git-codecommit.ap-northeast-2.amazonaws.com/v1/repos/mani'

        CONTAINER_REPO = '865577889736.dkr.ecr.ap-northeast-2.amazonaws.com/fast'
        CONTAINER_REGISTRY_CREDENTIAL = 'ecr_cre'
    }
    stages {
        stage('Checkout Github') {
            
            steps {
            //    slackSend (
            //        channel: '#dep02', 
            //        color: '#FFFF00', 
            //        message: "STARTED: ${currentBuild.number}"
            //    )

                  checkout scm
            }
            post {
                always {
                    sh "echo always"
                }
                failure {
                    sh "echo clone failed"
                }
                success {
                    sh "echo clone success"
                }
                
            }
        }
        stage('docker image build') {
            steps {
                sh "docker build -t ${CONTAINER_REPO}:${currentBuild.number} ."
                sh "docker build -t ${CONTAINER_REPO}:latest ."
                // currentBuild.number 젠킨스가 제공하는 빌드넘버 변수
                // oolralra/fast:<빌드넘버> 와 같은 이미지가 만들어질 예정.
                
            }
            post {
                failure {
                    sh "echo image build failed"
                }
                success {
                    sh "echo image build success"
                }
            }
        }
        stage('docker image push') {
            steps {
                withDockerRegistry(credentialsId: 'ecr:ap-northeast-2:ecr_cre', url: 'https://${CONTAINER_REPO}') {
                    sh "docker push ${CONTAINER_REPO}:${currentBuild.number}"
                    sh "docker push ${CONTAINER_REPO}:latest"
                }
            }
            post {
                failure {
                    sh "docker image rm -f ${CONTAINER_REPO}:${currentBuild.number}"
                    sh "docker image rm -f ${CONTAINER_REPO}:latest"
                    sh "echo push failed"
                    // 성공하든 실패하든 로컬에 있는 도커이미지는 삭제
                }
                success {
                    sh "docker image rm -f ${CONTAINER_REPO}:${currentBuild.number}"
                    sh "docker image rm -f ${CONTAINER_REPO}:latest"
                    sh "echo push success"
                    // 성공하든 실패하든 로컬에 있는 도커이미지는 삭제
                }
            }
        }
        stage('EKS manifest file update') {
            steps {
                git url: MANIFEST_REPO, branch: 'main'
                sh "git config --global user.email ${GIT_EMAIL}"
                sh "git config --global user.name ${GIT_NAME}"
                sh "sed -i 's@${CONTAINER_REPO}:.*@${CONTAINER_REPO}:${currentBuild.number}@g' fast.yml"

                sh "git add ."
                sh "git branch -M main"
                sh "git commit -m 'fixed tag ${currentBuild.number}'"
                sh "git remote remove origin"
                sh "git remote add origin ${MANIFEST_REPO}"
                sh "git push origin main"
            }
            post {
                failure {
                    sh "echo manifest update failed"
                }
                success {
                    sh "echo manifest update success"
                }
            }
        }
        

    }
}
