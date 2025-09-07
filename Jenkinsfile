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
        // These should point to your Jenkins credential IDs (string/secret text)
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')     // change to your credential id
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY') // change to your credential id
        AWS_DEFAULT_REGION = "${params.REGION}"
        KUBECONFIG = "${WORKSPACE}/kubeconfig"
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

        stage('Prepare kubeconfig (create in workspace)') {
            steps {
                // Use an image with aws cli (or ensure aws cli exists on agent)
                // Using docker is optional — if your agent has aws CLI you can run shell directly.
                script {
                    sh '''
                    echo ">>> aws version:"
                    aws --version || true

                    echo ">>> Creating kubeconfig for EKS cluster into ${KUBECONFIG}"
                    aws eks update-kubeconfig --name ${CLUSTER_NAME} --region ${REGION} --kubeconfig ${KUBECONFIG}

                    echo ">>> Validate kubeconfig file:"
                    ls -l ${KUBECONFIG}
                    grep -E "exec:|users:" ${KUBECONFIG} || true

                    echo ">>> Test kubectl connectivity"
                    kubectl --kubeconfig ${KUBECONFIG} get nodes || true
                    kubectl --kubeconfig ${KUBECONFIG} get pods -A || true
                    '''
                }
            }
        }

        stage('Helm Lint & Render (dry-run)') {
            steps {
                script {
                    sh '''#!/bin/bash
                    set -o pipefail
                    KUBECONFIG="${KUBECONFIG:-$WORKSPACE/kubeconfig}"
                    echo "Using kubeconfig: $KUBECONFIG"

                    # create namespace if not exists
                    if ! kubectl --kubeconfig "$KUBECONFIG" get ns "${CHART_PATH}" >/dev/null 2>&1; then
                    kubectl --kubeconfig "$KUBECONFIG" create ns "${CHART_PATH}"
                    fi

                    helm lint "${CHART_PATH}"
                    helm upgrade --install learner-report "${CHART_PATH}" \
                    -n "${CHART_PATH}" -f deployment/environments/values-dev.yaml \
                    --set frontend.image="${FRONTEND_IMAGE}" \
                    --set backend.image="${BACKEND_IMAGE}" \
                    --dry-run --debug
                    '''
                }
            }
        }


        stage('Deploy') {
            steps {
                sh '''
                helm upgrade --install learner-report ${CHART_PATH} \
                  -n ${CHART_PATH} -f deployment/environments/values-dev.yaml \
                  --set frontend.image=${FRONTEND_IMAGE} \
                  --set backend.image=${BACKEND_IMAGE}
                '''
            }
        }
    }

    post {
        failure {
            echo "Deployment failed — printing troubleshooting info:"
            sh '''
            kubectl --kubeconfig ${KUBECONFIG} get pods -A || true
            kubectl --kubeconfig ${KUBECONFIG} describe pods -A | sed -n '1,200p' || true
            '''
        }
    }
}
