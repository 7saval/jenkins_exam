pipeline {
    environment {
        dockerImageName = "geniusyoung/simple-echo"
    }
    agent {
        kubernetes {
            yaml '''
            apiVersion: v1
            kind: Pod
            spec:
                containers:
                - name: jnlp
                  image: sheayun/jnlp-agent-sample
                  env:
                  - name: DOCKER_HOST
                    value: "tcp://localhost:2375"
                - name: dind
                  image: docker:24-dind
                  command:
                  - /usr/local/bin/dockerd-entrypoint.sh
                  env:
                  - name: DOCKER_TLS_CERTDIR
                    value: ""
                  securityContext:
                    privileged: true
            '''
        }
    }
    stages {
        stage('git scm update') {
            steps {
                // sh "git clone https://github.com/7saval/jenkins_exam.git ."
                checkout scm
            }
        }
        stage('docker build && push') {
            // steps {
            //     // docker pipline 플러그인 안정화 시 사용 가능
            //     // script {
            //     //     dockerImage = docker.build.dockerImageName
            //     //     docker.withRegistry('https://registry.hub.docker.com',
            //     //         'dockerhub-credentials') {
            //     //             dockerImage.push("latest")
            //     //         }
            //     // }
            //     sh '''
            //       docker version
            //       docker build -t geniusyoung/simple-echo:latest .
            //       docker push geniusyoung/simple-echo:latest
            //     '''
            // }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASSWD'
                )]) {
                    sh '''
                      echo "$DOCKER_PASSWD" | docker login -u "$DOCKER_USER" --password-stdin
                      docker build -t geniusyoung/simple-echo:latest .
                      docker push geniusyoung/simple-echo:latest
                    '''
                }
            }
        }
        stage('deploy application on kubernetes cluster'){
            steps {
                withKubeConfig([credentialsId: 'KUBECONFIG',
                    serverUrl: 'https://kubernetes.default',
                    namespace: 'jenkins'
                ]) {
                    sh '''
                    kubectl apply -f deployment.yaml
                    kubectl apply -f service.yaml
                    '''
                }
            }
        }
    }
    post {
        always {
            sh 'docker logout'
        }
    }
}