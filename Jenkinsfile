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

/*
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
                        path: 'vault', // Đường dẫn bí mật trong Vault
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
*/

node { 
  def VAULT_ADDR = 'http://172.22.3.91:8200/'
  def VAULT_PATH_SSH = 'linux'
  def NEXUS_CRED = 'nexus_security'
  def NEXUS_ADDR = 'http://172.22.3.92:8081/'

  def secrets = [
          [
              path: "${VAULT_PATH_SSH}", engineVersion: 2,
              secretValues: [
                [vaultKey: 'windows_pass'], [vaultKey: 'windows_user'],
                [vaultKey: 'linux_pass'], [vaultKey: 'linux_user'],
              ]
          ]
  ]

  def configuration = [
        vaultUrl: "${VAULT_ADDR}",
        vaultCredentialId: 'd375013e-e3c3-42b8-a417-58b4dee65b99',
        engineVersion: 2
  ]   
  
  // Helper function to run ansible playbook
  def runAnsiblePlaybook(playbook, extraVars) {
    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: "${NEXUS_CRED}", usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASSWORD']]){
      withVault([configuration: configuration, vaultSecrets: secrets]) {
        sh """
        ansible-playbook -i host --extra-vars "${extraVars}" ${playbook}
        """
      }
    }
  }

  stage("Checkout SCM") {
    cleanWs()
    checkout scm
  }

  properties([
    parameters([
        [
            $class: 'ChoiceParameter',
            choiceType: 'PT_SINGLE_SELECT',
            description: 'Select Build Type',
            name: 'build_type',
            randomName: 'choice-parameter-5631314439613978',
            script: [
                $class: 'GroovyScript',
                fallbackScript: [
                    classpath: [],
                    sandbox: true,
                    script: '''return ["Could not get build type"]'''
                ],
                script: [
                    classpath: [],
                    sandbox: true,
                    script: '''return ["Verify", "Install", "Rollback"]'''
                ]
            ]
        ],
        [
            $class: 'CascadeChoiceParameter',
            choiceType: 'PT_CHECKBOX',
            description: 'Select PlayBook',
            filterLength: 1,
            filterable: true,
            name: 'ansiblePlaybook',
            randomName: 'choice-parameter-banca-5631314456178620',
            referencedParameters: 'build_type',
            script: [
                $class: 'GroovyScript',
                fallbackScript: [
                    classpath: [],
                    sandbox: true,
                    script: '''return ["Plz choose something in list"]'''
                ],
                script: [
                    classpath: [], 
                    sandbox: true, 
                    script: '''
                    if (build_type == "Verify") {
                        return ['check_connection_linux', 'check_connection_windows']
                    } else if (build_type == "Install") {
                        return ['install_linux_xdr', 'install_linux_prisma', 'install_windows_xdr', 'install_windows_prisma']
                    } else if (build_type == "Rollback") {
                        return ['rollback_linux_xdr', 'rollback_linux_prisma', 'rollback_windows_xdr', 'rollback_windows_prisma']
                    }
                    '''
                ]
            ]
        ]
    ])
])

def para_return = "${params.ansiblePlaybook}"
def listServices = para_return.split(',')
echo "Build Selected is: ${listServices}"

// Run checks for Linux and Windows
if (listServices.contains("check_connection_linux")) {
  runAnsiblePlaybook("check_connection_linux.yaml", "linux_user=${linux_user} linux_pass=${linux_pass} nexus_user=${NEXUS_USER} nexus_password=${NEXUS_PASSWORD}")
}
if (listServices.contains("check_connection_windows")) {
  runAnsiblePlaybook("check_connection_windows.yaml", "win_user=${windows_user} win_pass=${windows_pass} nexus_user=${NEXUS_USER} nexus_password=${NEXUS_PASSWORD}")
}

// ------------------ Linux XDR ------------------
if (listServices.contains("install_linux_xdr")) {
  runAnsiblePlaybook("install_linux_xdr.yaml", "linux_user=${linux_user} linux_pass=${linux_pass} nexus_user=${NEXUS_USER} nexus_password=${NEXUS_PASSWORD}")
}
if (listServices.contains("rollback_linux_xdr")) {
  runAnsiblePlaybook("rollback_linux_xdr.yaml", "linux_user=${linux_user} linux_pass=${linux_pass}")
}

// ------------------ Linux Prisma ------------------
if (listServices.contains("install_linux_prisma")) {
  runAnsiblePlaybook("install_linux_prisma.yaml", "linux_user=${linux_user} linux_pass=${linux_pass}")
}
if (listServices.contains("rollback_linux_prisma")) {
  runAnsiblePlaybook("rollback_linux_prisma.yaml", "linux_user=${linux_user} linux_pass=${linux_pass}")
}

// ------------------ Windows XDR ------------------
if (listServices.contains("install_windows_xdr")) {
  runAnsiblePlaybook("install_windows_xdr.yaml", "win_user=${windows_user} win_pass=${windows_pass} nexus_user=${NEXUS_USER} nexus_password=${NEXUS_PASSWORD}")
}
if (listServices.contains("rollback_windows_xdr")) {
  runAnsiblePlaybook("rollback_windows_xdr.yaml", "win_user=${windows_user} win_pass=${windows_pass}")
}

// ------------------ Windows Prisma ------------------
if (listServices.contains("install_windows_prisma")) {
  runAnsiblePlaybook("install_windows_prisma.yaml", "win_user=${windows_user} win_pass=${windows_pass}")
}
if (listServices.contains("rollback_windows_prisma")) {
  runAnsiblePlaybook("rollback_windows_prisma.yaml", "win_user=${windows_user} win_pass=${windows_pass}")
}
}