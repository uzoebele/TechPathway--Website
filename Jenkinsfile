pipeline {
    agent any

    environment {
        AWS_REGION  = 'us-east-1'
        AWS_ACCOUNT = '669977209932'
        REGISTRY    = "${AWS_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com"
    }

    stages {

        // ---------- 1. BUILD ----------
        stage('Build images') {
            steps {
                sh 'docker build -t action-messages:$BUILD_NUMBER       action-messages/'
                sh 'docker build -t techpathway-warehouse:$BUILD_NUMBER techpathway-warehouse/'
                sh 'docker build -t weekly-call:$BUILD_NUMBER           weekly-call/'
            }
        }

        // ---------- 2. SMOKE TEST ----------
        // Start each image, ask it if it is alive, then throw it away.
        stage('Smoke test') {
            steps {
                sh '''
                    set -e
                    docker rm -f smoke-am >/dev/null 2>&1 || true
                    docker run -d --name smoke-am action-messages:$BUILD_NUMBER
                    sleep 10
                    docker exec smoke-am python -c "import urllib.request; urllib.request.urlopen('http://localhost:5001/health')"
                    docker rm -f smoke-am
                '''
                sh '''
                    set -e
                    docker rm -f smoke-wh >/dev/null 2>&1 || true
                    docker run -d --name smoke-wh techpathway-warehouse:$BUILD_NUMBER
                    sleep 10
                    docker exec smoke-wh python -c "import urllib.request; urllib.request.urlopen('http://localhost:5002/health')"
                    docker rm -f smoke-wh
                '''
                sh '''
                    set -e
                    docker rm -f smoke-wc >/dev/null 2>&1 || true
                    docker run -d --name smoke-wc weekly-call:$BUILD_NUMBER
                    sleep 20
                    docker exec smoke-wc python -c "import urllib.request; urllib.request.urlopen('http://localhost:5111/health')"
                    docker rm -f smoke-wc
                '''
            }
        }

        // ---------- 3. PUSH TO ECR ----------
        // Switched off until the 'aws-creds' credential exists in Jenkins.
        // Turn back on by deleting the leading // on each line below.
        // stage('Push to ECR') {
        //     steps {
        //         withCredentials([usernamePassword(
        //             credentialsId: 'aws-creds',
        //             usernameVariable: 'AWS_ACCESS_KEY_ID',
        //             passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
        //             sh '''
        //                 set -e
        //                 aws ecr get-login-password --region $AWS_REGION \
        //                   | docker login --username AWS --password-stdin $REGISTRY
        //
        //                 for APP in action-messages techpathway-warehouse weekly-call; do
        //                     docker tag  $APP:$BUILD_NUMBER $REGISTRY/$APP:$BUILD_NUMBER
        //                     docker push $REGISTRY/$APP:$BUILD_NUMBER
        //                 done
        //             '''
        //         }
        //     }
        // }

        // ---------- 4. DEPLOY TO EKS ----------
        // Switched off until the cluster exists.
        // stage('Deploy to EKS') {
        //     steps {
        //         withCredentials([usernamePassword(
        //             credentialsId: 'aws-creds',
        //             usernameVariable: 'AWS_ACCESS_KEY_ID',
        //             passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
        //             sh '''
        //                 aws eks update-kubeconfig --name techpathway --region $AWS_REGION
        //                 kubectl apply -f kubernetes/
        //                 kubectl -n techpathway set image deployment/action-messages \
        //                     action-messages=$REGISTRY/action-messages:$BUILD_NUMBER
        //             '''
        //         }
        //     }
        // }

    }

    post {
        always {
            sh 'docker rm -f smoke-am smoke-wh smoke-wc >/dev/null 2>&1 || true'
            echo 'Pipeline finished.'
        }
    }
}
