/*
def NEXUS_CRED = 'nexus_security'

node {
    withCredentials([[$class: 'UsernamePasswordMultiBinding', 
      credentialsId: "${NEXUS_CRED}", 
      usernameVariable: 'NEXUS_USER', 
      passwordVariable: 'NEXUS_PASSWORD']]) {
        sh '''
          echo "Username: $NEXUS_USER"
          echo "Password Length: ${#NEXUS_PASSWORD}"
        '''
    }
}
*/
pipeline {
    agent any
    environment {
        VAULT_ADDR = 'http://172.22.3.91:8200'
        VAULT_CRED_ID = 'd375013e-e3c3-42b8-a417-58b4dee65b99' // ID trong Jenkins Credentials
    }
    stages {
        stage('Vault Test') {
            steps {
                withVault(configuration: [
                    vaultUrl: env.VAULT_ADDR,
                    vaultCredentialId: env.VAULT_CRED_ID
                ], vaultSecrets: [
                    [
                        path: 'secret/vault', // Đường dẫn bí mật trong Vault
                        secretValues: [
                            [envVar: 'MY_SECRET', vaultKey: 'username'] // Biến môi trường và khóa bí mật
                        ]
                    ]
                ]) {
                    sh 'echo $MY_SECRET' // In bí mật (chỉ để kiểm tra, cẩn thận với log)
                    echo 'Vault connection successful!'
                }
            }
        }
    }
}