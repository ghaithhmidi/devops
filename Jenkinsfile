pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "ghaithhmidi/devops"
        DOCKER_TAG = "latest"
        NAMESPACE = "devops"
    }

    stages {

        stage('Git Checkout') {
            steps {
                echo 'Code already checked out by Jenkins'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker build -t $DOCKER_IMAGE:$DOCKER_TAG .
                        docker push $DOCKER_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }

        stage('Kubernetes Deploy') {
            steps {
                sh '''
                    kubectl apply -f k8s/pv-sql.yaml -n $NAMESPACE
                    kubectl apply -f k8s/pvc-sql.yaml -n $NAMESPACE
                    kubectl apply -f k8s/deploy-sql.yaml -n $NAMESPACE
                    kubectl apply -f k8s/service-sql.yaml -n $NAMESPACE
                    kubectl apply -f k8s/configmap-spring.yaml -n $NAMESPACE
                    kubectl apply -f k8s/secret-spring.yaml -n $NAMESPACE
                    kubectl apply -f k8s/deploy-spring.yaml -n $NAMESPACE
                    kubectl apply -f k8s/service-spring.yaml -n $NAMESPACE
                '''
            }
        }

    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}