pipeline {
    agent any

    environment {
        VAULT_ADDR = 'http://vault:8200'  // Your Vault address
    }

    stages {
        stage('Read secret from Vault') {
            steps {
                withCredentials([string(credentialsId: 'vault-jenkins-token', variable: 'VAULT_TOKEN')]) {
                    script {
                        // Get AWS key and secret from Vault
                        def vaultData = sh(
                            script: """
                            curl -s --header "X-Vault-Token:${VAULT_TOKEN}" --request GET \
                            ${VAULT_ADDR}/v1/jenkins/my-secret/data/jenkins/my-secret
                            """,
                            returnStdout: true
                        ).trim()

                        def json = readJSON text: vaultData
                        env.AWS_ACCESS_KEY_ID = json.data.data.aws_access_key_id
                        env.AWS_SECRET_ACCESS_KEY = json.data.data.aws_secret_access_key
                        env.AWS_REGION = 'il-central-1'  // or read from Vault too
                    }
                }
            }
        }
            stage('Print EC2 Instance Names') {
                steps {
                    script {
                        withEnv([
                            "AWS_ACCESS_KEY_ID=${env.AWS_ACCESS_KEY_ID}",
                            "AWS_SECRET_ACCESS_KEY=${env.AWS_SECRET_ACCESS_KEY}",
                            "AWS_REGION=${env.AWS_REGION}"
                        ]) {
                            sh '''
                                echo "Using AWS key: $AWS_ACCESS_KEY_ID"
                                echo "Fetching EC2 instance names..."
                                aws ec2 describe-instances \
                                --region $AWS_REGION \
                                --query 'Reservations[*].Instances[*].Tags[?Key==`Name`].Value[]' \
                                --output text
                            '''
                        }
                    }
                }
            }
    }
}