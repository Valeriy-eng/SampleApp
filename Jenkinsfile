def buildTag = ''
def imageName = 'sampleapp'

// Reusable function to build and push Docker image
def buildAndPushDockerImage(tag) {
    withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
        sh """
            docker build -t ${imageName}:${tag} .
            echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
            docker tag ${imageName}:${tag} ${DOCKER_USER}/${imageName}:${tag}
            docker push ${DOCKER_USER}/${imageName}:${tag}
        """
    }
}

// Reusable function to deploy to Kubernetes
def deployToKubernetes(tag, namespace) {
    withCredentials([file(credentialsId: 'kubeconfig-creds', variable: 'KUBECONFIG')]) {
        sh """
            sed -i 's|IMAGE_TAG|${tag}|g' deployment.yaml
            sed -i 's|NAMESPACE_PLACEHOLDER|${namespace}|g' deployment.yaml
            kubectl apply -n ${namespace} -f deployment.yaml
        """
    }
}

pipeline {
    agent { label 'deploy' }

    parameters {
        string(name: 'KUBE_NAMESPACE', defaultValue: 'default', description: 'Kubernetes namespace to deploy to')
    }

    environment {
        IMAGE_NAME = 'sampleapp'
    }

    stages {

        stage('Initialize Build Tag') {
            steps {
                script {
                    def date = new Date().format("yyyyMMdd")
                    buildTag = "${date}.${env.BUILD_NUMBER}"
                    currentBuild.displayName = "Build-${buildTag}"
                    echo "Generated build tag: ${buildTag}"
                }
            }
        }

        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/Valeriy-eng/SampleApp.git', branch: 'master'
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    buildAndPushDockerImage(buildTag)
                }
            }
        }

        stage('Confirm Deployment') {
            steps {
                script {
                    input message: "Proceed to deploy ${buildTag} to namespace '${params.KUBE_NAMESPACE}'?", ok: 'Deploy'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    deployToKubernetes(buildTag, params.KUBE_NAMESPACE)
                }
            }
        }
    }
}
