pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        TF_VAR_aws_region = "${AWS_DEFAULT_REGION}"
        KUBECONFIG = "${WORKSPACE}/kubeconfig"
        TERRAFORM_VERSION = "1.1.0"
    }

    stages {
        stage('Preparation') {
            steps {
                script {
                    // Install Terraform
                    sh """
                    wget https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip
                    unzip terraform_${TERRAFORM_VERSION}_linux_amd64.zip
                    sudo mv terraform /usr/local/bin/
                    """

                    // Install kubectl
                    sh """
                    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                    chmod +x ./kubectl
                    sudo mv ./kubectl /usr/local/bin/kubectl
                    """
                }
            }
        }

        stage('Init') {
            steps {
                dir('eks'){
                    sh 'terraform init'
                }
            }
        }

        stage('Apply Terraform') {
            steps {
                dir('eks'){
                    sh 'terraform apply -auto-approve'
                }
            }
        }

        stage('Configure kubectl') {
            steps {
                script {
                    EKS_CLUSTER_NAME = sh(script: 'terraform output -raw cluster_id', returnStdout: true).trim()
                    sh "aws eks --region ${AWS_DEFAULT_REGION} update-kubeconfig --name ${EKS_CLUSTER_NAME} --kubeconfig ${KUBECONFIG}"
                }
            }
        }

        stage('Deploy Nginx') {
            steps {
                sh "kubectl --kubeconfig=${KUBECONFIG} apply -f nginx/nginx-deployment.yaml"
            }
        }
    }
}
