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
                        def response = sh(
                            script: """
                            curl -s --header "X-Vault-Token:${vault_token}" --request GET  http://vault:8200/v1/jenkins/my-secret/data/jenkins/my-secret | jq '.data.data.key'
                            """,
                            returnStdout: true
                            ).trim()


                      
                    }
                }
            }
        }
    }
}