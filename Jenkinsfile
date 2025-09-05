pipeline {
    agent any

    parameters {
        string(name: 'CLUSTER_NAME', defaultValue: 'k-mern-clus', description: 'EKS cluster name')
        string(name: 'REGION', defaultValue: 'us-west-2', description: 'AWS region')
        string(name: 'CHART_PATH', defaultValue: 'mern-chart', description: 'Path to Helm chart in workspace')
        string(name: 'FRONTEND_IMAGE', defaultValue: 'noizy23yo/learner-report-frontend:latest', description: 'Frontend image tag')
        string(name: 'BACKEND_IMAGE', defaultValue: 'noizy23yo/learner-report-backend:latest', description: 'Backend image tag')
    }
    environment {
        KUBECONFIG_CRED_ID = 'kubeconfig'     
        MONGO_URI_CRED_ID = 'MONGO_URI'       
        HASH_KEY_CRED_ID = 'HASH_KEY'         
        JWT_SECRET_CRED_ID = 'JWT_SECRET'     
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
                    export AWS_DEFAULT_REGION=us-west-2

                    echo ">>> Updating kubeconfig"
                    aws eks update-kubeconfig --name k-mern-clus --region us-west-2

                    echo ">>> Cluster Info"
                    kubectl get nodes
                    kubectl get pods -A
                    '''
                }
            }
        }

        stage('Helm Lint & Render (dry-run)') {
            steps {
                withCredentials([file(credentialsId: env.KUBECONFIG_CRED_ID, variable: 'KUBECONFIG_FILE')]) {
                sh '''
                    export KUBECONFIG="$KUBECONFIG_FILE"
                    helm version
                    helm lint ${CHART_PATH} || true
                    # Do a dry-run render to catch templating errors
                    helm template --values ${CHART_PATH}/values.yaml ${CHART_PATH} --set frontend.image=${FRONTEND_IMAGE} --set backend.image=${BACKEND_IMAGE} --namespace ${NAMESPACE} --debug > /tmp/helm-template-output.yaml
                    echo "Helm template rendered to /tmp/helm-template-output.yaml"
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
