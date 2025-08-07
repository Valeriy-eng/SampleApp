def buildTag = ''

// Reusable stage function for checkout
def checkoutCode() {
    stage('Checkout Code') {
        git url: 'https://github.com/Valeriy-eng/SampleApp.git', branch: 'master'
        echo "Checked out code from Git repository"
    }
}

// Reusable stage function for Docker build & push
def buildAndPushDockerImage(tag) {
    stage('Build & Push Docker Image') {
        withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
            bat """
                docker build -t sampleapp:${tag} .
                echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                docker tag sampleapp:${tag} %DOCKER_USER%/sampleapp:${tag}
                docker push %DOCKER_USER%/sampleapp:${tag}
            """
        }
        echo "Docker image built and pushed with tag: ${tag}"
    }
}

// Reusable stage function for deployment
def deployToKubernetes(tag, namespace) {
    stage('Deploy to Kubernetes') {
        input message: "Proceed with Kubernetes deployment to namespace '${namespace}'?", ok: 'Deploy'
        withCredentials([file(credentialsId: 'kubeconfig-creds', variable: 'KUBECONFIG')]) {
            bat """
                powershell -Command "(Get-Content deployment.yaml) -replace 'IMAGE_TAG', '${tag}' -replace 'NAMESPACE', '${namespace}' | Set-Content deployment.yaml"
                kubectl apply -f deployment.yaml --namespace=${namespace}
            """
        }
        echo "Deployed to Kubernetes namespace: ${namespace} with image tag: ${tag}"
    }
}

pipeline {
    agent { label 'deploy' }

    parameters {
        string(name: 'NAMESPACE', defaultValue: 'default', description: 'Kubernetes namespace to deploy to')
    }

    stages {
        stage('Generate Tag') {
            steps {
                script {
                    def date = new Date().format('yyyyMMdd')
                    buildTag = "${date}.${env.BUILD_NUMBER}"
                    currentBuild.displayName = buildTag
                    echo "Generated build tag: ${buildTag}"
                }
            }
        }

        stage('Show Build Tag') {
            steps {
                echo "Current build tag is: ${buildTag}"
            }
        }

        // Use reusable stages
        stage('Run Pipeline Stages') {
            steps {
                script {
                    checkoutCode()
                    buildAndPushDockerImage(buildTag)
                    deployToKubernetes(buildTag, params.NAMESPACE)
                }
            }
        }
    }
}
