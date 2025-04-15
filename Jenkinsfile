pipeline {
    agent any

    environment {
        ACR_NAME           = 'dipeshacr01'
        AZURE_CREDENTIALS_ID = 'jenkins-pipeline-sp' // You can remove this if not used
        ACR_LOGIN_SERVER   = 'dipeshacr01.azurecr.io'
        IMAGE_NAME         = 'webapidocker1'
        IMAGE_TAG          = 'latest'
        RESOURCE_GROUP     = 'myResourceGroup'
        AKS_CLUSTER        = 'myAKSCluster'
        TF_WORKING_DIR     = '.'

        ARM_CLIENT_ID       = 'cf9ec97f-17eb-4f0e-93ea-84d1b95e9c1e'
        ARM_CLIENT_SECRET   = 'ZjI8Q~5YIhoDiUQ~QTtoHan88ai0recw2LN_sbUk'
        ARM_TENANT_ID       = '2c1fe611-7a02-40df-ad5d-f13e1900b40b'
        ARM_SUBSCRIPTION_ID = '1f269429-e833-4270-8df5-e6b97ba7a20c'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/dipeshagarwaaal/Jenkins-Docker-Kubernetes-Terraform.git'
            }
        }

        stage('Build .NET App') {
            steps {
                bat 'dotnet publish WebApiJenkins/WebApiJenkins.csproj -c Release -o out'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t %ACR_LOGIN_SERVER%/%IMAGE_NAME%:%IMAGE_TAG% -f WebApiJenkins/Dockerfile WebApiJenkins"
            }
        }

        stage('Terraform Init') {
            steps {
                withEnv([
                    "ARM_CLIENT_ID=${env.ARM_CLIENT_ID}",
                    "ARM_CLIENT_SECRET=${env.ARM_CLIENT_SECRET}",
                    "ARM_SUBSCRIPTION_ID=${env.ARM_SUBSCRIPTION_ID}",
                    "ARM_TENANT_ID=${env.ARM_TENANT_ID}"
                ]) {
                    bat """
                    cd %TF_WORKING_DIR%
                    terraform init
                    """
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                withEnv([
                    "ARM_CLIENT_ID=${env.ARM_CLIENT_ID}",
                    "ARM_CLIENT_SECRET=${env.ARM_CLIENT_SECRET}",
                    "ARM_SUBSCRIPTION_ID=${env.ARM_SUBSCRIPTION_ID}",
                    "ARM_TENANT_ID=${env.ARM_TENANT_ID}"
                ]) {
                    bat """
                    cd %TF_WORKING_DIR%
                    terraform plan -out=tfplan
                    """
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                withEnv([
                    "ARM_CLIENT_ID=${env.ARM_CLIENT_ID}",
                    "ARM_CLIENT_SECRET=${env.ARM_CLIENT_SECRET}",
                    "ARM_SUBSCRIPTION_ID=${env.ARM_SUBSCRIPTION_ID}",
                    "ARM_TENANT_ID=${env.ARM_TENANT_ID}"
                ]) {
                    bat """
                    cd %TF_WORKING_DIR%
                    terraform apply -auto-approve tfplan
                    """
                }
            }
        }

        stage('Login to ACR') {
            steps {
                bat "az acr login --name %ACR_NAME%"
            }
        }

        stage('Push Docker Image to ACR') {
            steps {
                bat "docker push %ACR_LOGIN_SERVER%/%IMAGE_NAME%:%IMAGE_TAG%"
            }
        }

        stage('Get AKS Credentials') {
            steps {
                bat "az aks get-credentials --resource-group %RESOURCE_GROUP% --name %AKS_CLUSTER% --overwrite-existing"
            }
        }

        stage('Deploy to AKS') {
            steps {
                bat "kubectl apply -f WebApiJenkins/test.yaml"
            }
        }
    }

    post {
        success {
            echo '✅ All stages completed successfully!'
        }
        failure {
            echo '❌ Build failed.'
        }
    }
}
