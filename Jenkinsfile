pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = 'codrin2'
        GITHUB_URL = 'https://github.com/KEA-4th-TEAM-BOOT/pipeline-config.git'

    }

    stages {
        stage('소스파일 체크아웃') {
            steps {
                // 소스코드를 가져올 Github 주소
                git branch: 'main', url: 'https://github.com/KEA-4th-TEAM-BOOT/nginx-exercise.git'
            }
        }



        stage('컨테이너 빌드') {
            steps {
                // 도커 빌드
                sh "docker build -t ${DOCKERHUB_USERNAME}/pipeline-test:v1.0.0 ."
            }
        }

        stage('컨테이너 업로드') {
            steps {
                // DockerHub로 이미지 업로드
                sh "docker push ${DOCKERHUB_USERNAME}/pipeline-test:v1.0.0"

            }
        }

        stage('릴리즈파일 체크아웃') {
            steps {
                checkout scmGit(branches: [[name: '*/main']],
                                userRemoteConfigs: [[url: "${GITHUB_URL}"]])
            }
        }


        stage('쿠버네티스 배포') {
            steps {
                // K8S 배포
                sh "kubectl apply -f ./deploy/k8s/namespace.yaml"
                sh "kubectl apply -f ./deploy/k8s/configmap.yaml"
                sh "kubectl apply -f ./deploy/k8s/secret.yaml"
                sh "kubectl apply -f ./deploy/k8s/service.yaml"
                sh "kubectl apply -f ./deploy/k8s/deployment.yaml"
            }
        }

        stage('배포 확인') {
            steps {
                // 10초 대기
                sh "sleep 10"

                // K8S 배포
                sh "kubectl get -f ./deploy/k8s/namespace.yaml"
                sh "kubectl get -f ./deploy/k8s/configmap.yaml"
                sh "kubectl get -f ./deploy/k8s/secret.yaml"
                sh "kubectl get -f ./deploy/k8s/service.yaml"
                sh "kubectl get -f ./deploy/k8s/deployment.yaml"
            }
        }
    }
}