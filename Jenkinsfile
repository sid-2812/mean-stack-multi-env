pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/sid-2812/mean-stack-multi-env.git'
            }
        }

        stage('Build') {
            steps {
                dir('client') {
                    sh 'npm install'
                    sh 'npm run build'
                }

                dir('server') {
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
        }

        stage('Test') {
            steps {
                dir('server') {
                    sh 'npm test'
                    sh 'npm run test:integration'
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t siddharth2812/mean-client:${BUILD_NUMBER} ./client'
                sh 'docker build -t siddharth2812/mean-server:${BUILD_NUMBER} ./server'
            }
        }

       
        stage('Trivy Scan') {
            steps {
                sh '''
                    set +e

                    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v trivy-cache:/root/.cache/ aquasec/trivy:0.76.1 image --scanners vuln --severity HIGH,CRITICAL --exit-code 1 siddharth2812/mean-client:${BUILD_NUMBER}
                    CLIENT_SCAN=$?

                    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v trivy-cache:/root/.cache/ aquasec/trivy:0.76.1 image --scanners vuln --severity HIGH,CRITICAL --exit-code 1 siddharth2812/mean-server:${BUILD_NUMBER}
                    SERVER_SCAN=$?

                    if [ "$CLIENT_SCAN" -ne 0 ] || [ "$SERVER_SCAN" -ne 0 ]; then
                        echo "Trivy security gate failed: check HIGH/CRITICAL vulnerabilities."
                        exit 1
                    fi
                '''
            }
        }


        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials', 
                        usernameVariable: 'DOCKERHUB_USERNAME', 
                        passwordVariable: 'DOCKERHUB_TOKEN'
                        )
                        ]) {
                    sh 'echo $DOCKERHUB_TOKEN | docker login -u $DOCKERHUB_USERNAME --password-stdin'
                    sh 'docker push siddharth2812/mean-client:${BUILD_NUMBER}'
                    sh 'docker push siddharth2812/mean-server:${BUILD_NUMBER}'
                    sh 'docker logout'
                }
            }
        }
    }
}