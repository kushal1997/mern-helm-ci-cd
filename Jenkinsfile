pipeline {
    agent any

    parameters {
        string(name: 'FRONTEND_IMAGE', defaultValue: 'noizy23yo/learner-report-frontend:latest', description: 'Frontend image tag')
        string(name: 'BACKEND_IMAGE', defaultValue: 'noizy23yo/learner-report-backend:latest', description: 'Backend image tag')
    }

    stages {
        stage('Checkout Helm Repo') {
            steps {
                git 'https://github.com/kushal1997/mern-helm-ci-cd.git'
            }
        }

        stage('Verify K8s Access') {
            steps {
                withCredentials([
                    string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh '''
                    export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                    export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                    export AWS_DEFAULT_REGION=us-west-2

                    echo ">>> Updating kubeconfig"
                    aws eks update-kubeconfig --name k-mern-cluster --region us-west-2

                    echo ">>> Cluster Info"
                    kubectl get nodes
                    kubectl get pods -A
                    '''
                }
            }
        }

        stage('Deploy with Helm') {
            steps {
                withCredentials([
                    string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh '''
                    export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                    export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                    export AWS_DEFAULT_REGION=us-west-2

                    aws eks update-kubeconfig --name k-mern-cluster --region us-west-2

                    echo ">>> Deploying MERN app"
                    helm upgrade --install mern-app ./mern-chart \
                      --set frontend.image=$FRONTEND_IMAGE \
                      --set backend.image=$BACKEND_IMAGE

                    echo ">>> Post-deploy check"
                    kubectl get pods
                    kubectl get svc
                    kubectl get ingress
                    '''
                }
            }
        }
    }
}
