pipeline {
    agent any

    environment {
        VAULT_ADDR = 'http://vault:8200'
    }

    stages {
        stage('Read secret from Vault') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'vault-jenkins-token', variable: 'VAULT_TOKEN')]) {
                        def vaultData = sh(
                            script: """
                            curl -s --header "X-Vault-Token:${VAULT_TOKEN}" --request GET \
                            ${VAULT_ADDR}/v1/jenkins/my-secret/data/jenkins/my-secret
                            """,
                            returnStdout: true
                        ).trim()

                        echo "Vault response: ${vaultData}"

                        def json = readJSON text: vaultData

                        
                        def awsKey = json.data.data.aws_access_key_id
                        def awsSecret = json.data.data.aws_secret_access_key
                        def awsRegion = 'il-central-1'

                        
                        writeFile file: 'aws_env.sh', text: """
                            export AWS_ACCESS_KEY_ID=${awsKey}
                            export AWS_SECRET_ACCESS_KEY=${awsSecret}
                            export AWS_REGION=${awsRegion}
                        """
                    }
                }
            }
        }

        stage('Print EC2 Instance Names') {
            steps {
                script {
                 
                    def awsEnv = readFile('aws_env.sh').split("\n").findAll { it }
                    withEnv(awsEnv) {
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
