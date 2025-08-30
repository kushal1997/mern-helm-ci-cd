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

        stage('Deploy with Helm') {
            steps {
                sh """
                helm upgrade --install mern-app ./mern-chart \
                  --set frontend.image=${params.FRONTEND_IMAGE} \
                  --set backend.image=${params.BACKEND_IMAGE}
                """
            }
        }
    }
}
