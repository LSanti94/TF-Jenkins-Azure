pipeline {
    parameters {
        booleanParam(name: 'autoApprove', defaultValue: false, description: 'Automatically run apply after generating plan?')
    }
    environment {
        AZURE_CREDENTIALS_ID = 'AzureServicePrincipal'
/*      ARM_CLIENT_ID = credentials('AZURE_CREDENTIALS_ID').clientId
        ARM_CLIENT_SECRET = credentials('AZURE_CREDENTIALS_ID').clienSecretId
        ARM_SUBSCRIPTION_ID = credentials('AZURE_CREDENTIALS_ID').subcriptionId
        ARM_TENANT_ID = credentials('AZURE_CREDENTIALS_ID').tenantId*/
        ARM_ENVIRONMENT = 'public'
    }
    agent any
    stages {
        stage('Prepare') {
            steps {
                script {
                    def azureCreds = credentials('AzureServicePrincipal')
                    env.ARM_CLIENT_ID = azureCreds.clientId
                    env.ARM_CLIENT_SECRET = azureCreds.clientSecret
                    env.ARM_SUBSCRIPTION_ID = azureCreds.subscriptionId
                    env.ARM_TENANT_ID = azureCreds.tenantId
                } catch (Exception e) {
                      error("Failed to load Azure credentials: ${e.message}")
                }
            }
        }
        stage('checkout') {
            steps {
                script {
                    dir('terraform') {
                        git "https://github.com/LSanti94/TF-Jenkins-Azure.git"
                    }
                }
            }
        }

        stage('Plan') {
            steps {
                sh 'pwd; cd terraform/ ; terraform init'
                sh 'pwd; cd terraform/ ; terraform plan -out tfplan'
                sh 'pwd; cd terraform/ ; terraform show -no-color tfplan > tfplan.txt'
            }
        }

        stage('autoApprove') {
            when {
                not {
                    equals expected: true, actual: params.autoApprove
                }
            }
            steps {
                script {
                    def plan = readFile 'terraform/tfplan.txt'
                    input message: "Do you want to apply the plan?"
                    parameters: [text(name: 'Plan', description: 'Please review the plan', defaultValue: plan)]
                }
            }
        }

        stage('Apply') {
            when {
                expression { params.autoApprove == true }
            }
            steps {
                sh 'pwd; cd terraform/ ; terraform apply -input=false tfplan'
            }
        }

/*        stage('Destroy') {
            steps {
                sh 'pwd; cd terraform/ ; terraform destroy -auto-approve'
            }
        }*/
    }
}

