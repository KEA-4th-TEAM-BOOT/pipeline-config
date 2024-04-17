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



        stage('릴리즈파일 체크아웃') {
            steps {
                checkout scmGit(branches: [[name: '*/main']],
                                userRemoteConfigs: [[url: "${GITHUB_URL}"]])
            }
        }

        stage('컨테이너 빌드') {
            steps {
                // 도커 빌드
                sh "docker build -t ${DOCKERHUB_USERNAME}/pipeline-test:v1.0.0 ./build/docker"
            }
        }

        stage('컨테이너 업로드') {
            steps {
                // DockerHub로 이미지 업로드
                script{
                    if (DOCKERHUB_USERNAME == "codrin2") {
                        echo "docker push ${DOCKERHUB_USERNAME}/pipeline-test:v1.0.0"
                    } else {
                        sh "docker push ${DOCKERHUB_USERNAME}/pipeline-test:v1.0.0"
                    }
                }
            }
        }


        stage('쿠버네티스 Blue배포') {
            steps {
                sh "kubectl apply -f ./deploy/k8s/blue/namespace.yaml"
                sh "kubectl apply -f ./deploy/k8s/blue/configmap.yaml"
                sh "kubectl apply -f ./deploy/k8s/blue/secret.yaml"
                sh "kubectl apply -f ./deploy/k8s/blue/service.yaml"
                sh "kubectl apply -f ./deploy/k8s/blue/deployment.yaml"
            }
        }

        stage('자동배포 시작') {
            steps {
                input message: '자동배포 시작', ok: "Yes"
            }
        }

        stage('쿠버네티스 Green배포') {
            steps {
                sh "kubectl apply -f ./deploy/k8s/green/deployment.yaml"
            }
        }

        stage('Green 배포 확인중') {
            steps {
                script {
                    def returnValue
                    while (returnValue != "true true"){
                        returnValue = sh(returnStdout: true, encoding: 'UTF-8', script: "kubectl get -n pipelinetest-418 pods -l instance='api-tester-2214',blue-green-no='2' -o jsonpath='{.items[*].status.containerStatuses[*].ready}'")
                        echo "${returnValue}"
                        sleep 5
                    }
                }
            }
        }

        stage('Green 전환 완료') {
            steps {
                sh "kubectl patch -n pipelinetest-418 svc api-tester-2214 -p '{\"spec\": {\"selector\": {\"blue-green-no\": \"2\"}}}'"
            }
        }

        stage('Blue 삭제') {
            steps {
                sh "kubectl delete -f ./deploy/k8s/blue/deployment.yaml"
                sh "kubectl patch -n pipelinetest-418 svc api-tester-2214 -p '{\"metadata\": {\"labels\": {\"version\": \"2.0.0\"}}}'"
                sh "kubectl patch -n pipelinetest-418 cm api-tester-2214-properties -p '{\"metadata\": {\"labels\": {\"version\": \"2.0.0\"}}}'"
                sh "kubectl patch -n pipelinetest-418 secret api-tester-2214-postgresql -p '{\"metadata\": {\"labels\": {\"version\": \"2.0.0\"}}}'"
            }
        }
    }
}