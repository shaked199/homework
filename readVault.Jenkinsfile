pipeline {
    agent any

    environment {
        VAULT_ADDR = 'http://vault:8200'
        VAULT_SECRET_PATH = '/v1/jenkins/my-secret/data/jenkins/my-secret'  // <- fixed!
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Read secret from Vault') {
            steps {
                withCredentials([string(credentialsId: 'vault-jenkins-token', variable: 'VAULT_TOKEN')]) {
                    script {
                        def response = sh(
                            script: """curl -s --header "X-Vault-Token: ${VAULT_TOKEN}" --request GET ${VAULT_ADDR}/v1/${VAULT_SECRET_PATH}""",
                            returnStdout: true
                        ).trim()

                        echo "Vault response: ${response}"

                        def json = readJSON text: response
                        def secrets = json.data.data

                        writeFile file: 'aws_access_key_id.txt', text: secrets.aws_access_key_id
                        writeFile file: 'aws_secret_access_key.txt', text: secrets.aws_secret_access_key
                    }
                }
            }
        }

        stage('Print EC2 Instance Names') {
            steps {
                script {
                    def awsKey = readFile('aws_access_key_id.txt').trim()
                    def awsSecret = readFile('aws_secret_access_key.txt').trim()

                    withEnv([
                        "AWS_ACCESS_KEY_ID=${awsKey}",
                        "AWS_SECRET_ACCESS_KEY=${awsSecret}",
                        "AWS_REGION=il-central-1"
                    ]) {
                        sh '''
                            echo Using AWS key: $AWS_ACCESS_KEY_ID
                            echo Fetching EC2 instance names...
                            aws ec2 describe-instances \
                                --query "Reservations[*].Instances[*].Tags[?Key=='Name'].Value[]" \
                                --output text
                        '''
                    }
                }
            }
        }
    }
}
