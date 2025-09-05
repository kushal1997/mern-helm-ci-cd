pipeline {
    agent any

    parameters {
        string(name: 'CLUSTER_NAME', defaultValue: 'k-mern-clus', description: 'EKS cluster name')
        string(name: 'REGION', defaultValue: 'us-west-2', description: 'AWS region')
        string(name: 'FRONTEND_IMAGE', defaultValue: 'noizy23yo/learner-report-frontend:latest', description: 'Frontend image tag')
        string(name: 'BACKEND_IMAGE', defaultValue: 'noizy23yo/learner-report-backend:latest', description: 'Backend image tag')
    }

    stages {
        stage('Checkout Helm Repo') {
            steps {
                git branch: 'testing', url: 'https://github.com/kushal1997/mern-helm-ci-cd.git'
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
                    export AWS_DEFAULT_REGION=${REGION}

                    echo ">>> Confirm caller identity"
                    aws sts get-caller-identity

                    echo ">>> Updating kubeconfig for ${CLUSTER_NAME}"
                    aws eks update-kubeconfig --name ${CLUSTER_NAME} --region ${REGION}

                    echo ">>> Kube context"
                    kubectl config current-context

                    echo ">>> Cluster nodes & pods (all namespaces)"
                    kubectl get nodes -o wide
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
                    export AWS_DEFAULT_REGION=${REGION}

                    aws eks update-kubeconfig --name ${CLUSTER_NAME} --region ${REGION}

                    echo ">>> Deploying MERN app via Helm (will wait for resources to be ready)"
                    helm repo update || true
                    helm upgrade --install mern-app ./mern-chart \
                      --set frontend.image=${FRONTEND_IMAGE} \
                      --set backend.image=${BACKEND_IMAGE} \
                      --set ingress.enabled=true \
                      --set ingress.className=nginx \
                      --wait --timeout 10m

                    echo ">>> Post-deploy checks"
                    kubectl get pods -o wide
                    kubectl get svc
                    kubectl get ingress

                    # If you have deployments named frontend/backend, wait for their rollout:
                    kubectl rollout status deployment/frontend --timeout=120s || true
                    kubectl rollout status deployment/backend --timeout=120s || true

                    echo ">>> Done"
                    '''
                }
            }
        }
    }

    post {
        failure {
            echo "Deployment failed — printing troubleshooting info:"
            sh '''
            kubectl get pods -A
            kubectl describe pods -A | sed -n '1,200p'
            '''
        }
    }
}
