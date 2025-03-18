pipeline {
    parameters {
        booleanParam(name: 'autoApprove', defaultValue: false, description: 'Automatically run apply after generating plan?')
    }
    environment {
        AZURE_CLIENT_ID       = credentials('Azure_Client_ID')
        AZURE_CLIENT_SECRET   = credentials('Azure_Client_Secret')
        AZURE_TENANT_ID       = credentials('Azure_Tenant_ID')
/*        AZURE_SUBSCRIPTION_ID = credentials('Azure_Subscription_ID')*/
    }
    agent any
    stages {
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
