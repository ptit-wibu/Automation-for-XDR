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
/*
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
                    script:
                        'return[\'Could not get build type\']'
                ],
                script: [
                    classpath: [],
                    sandbox: true,
                    script:
                        '''
                        return ["Verify", "Install", "Rollback"]
                        '''
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
                    script:
                        'return[\'Plz choose something in list\']'
                ],
                script: [
                    classpath: [], 
                    sandbox: true, 
                    script: '''
                    if ( build_type == "Verify"){
                        return [
                                'check_connection_linux',
                                'check_connection_windows'
                                ]
                    } else if (build_type == "Install") {
                        return [
                                'install_linux_xdr',
                                'install_linux_prisma',
                                'install_windows_xdr',
                                'install_windows_prisma'
                                ]
                    } else if (build_type == "Rollback") {
                        return [
                                'rollback_linux_xdr',
                                'rollback_linux_prisma',
                                'rollback_windows_xdr',
                                'rollback_windows_prisma'
                                ]
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

if (listServices.contains("check_connection_linux")){
  withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: "${NEXUS_CRED}", usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASSWORD']]){
    withVault([configuration: configuration, vaultSecrets: secrets]) {
      stage("Check connection for Linux") {
        sh """
        ansible-playbook -i host --extra-vars "linux_user=${linux_user} linux_pass=${linux_pass} nexus_user=${NEXUS_USER} nexus_password=${NEXUS_PASSWORD}" check_connection_linux.yaml
        """
      }
    }
  }
}
if (listServices.contains("check_connection_windows")){
  withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: "${NEXUS_CRED}", usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASSWORD']]){
    withVault([configuration: configuration, vaultSecrets: secrets]) {
      stage("Check connection for Windows") {
        sh """
        ansible-playbook -i host --extra-vars "win_user=${windows_user} win_pass=${windows_pass} nexus_user=${NEXUS_USER} nexus_password=${NEXUS_PASSWORD}" check_connection_windows.yaml
        """
      }
    }
  }
}

// ------------------ Linux XDR ------------------
if (listServices.contains("install_linux_xdr")){
  withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: "${NEXUS_CRED}", usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASSWORD']]){
    withVault([configuration: configuration, vaultSecrets: secrets]) {
      stage("Install Linux XDR Agent") {
        sh """
        ansible-playbook -i host --extra-vars "linux_user=${linux_user} linux_pass=${linux_pass} nexus_user=${NEXUS_USER} nexus_password=${NEXUS_PASSWORD}" install_linux_xdr.yaml
        """
      }
    }
  }
}

if (listServices.contains("rollback_linux_xdr")){
  withVault([configuration: configuration, vaultSecrets: secrets]) {
    stage("Rollback Linux XDR") {
      sh """
      ansible-playbook -i host --extra-vars "linux_user=${linux_user} linux_pass=${linux_pass}" rollback_linux_xdr.yaml
      """
    }
  }
}

// ------------------ Linux Prisma ------------------
if (listServices.contains("install_linux_prisma")){
  withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: "${NEXUS_CRED}", usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASSWORD']]){
    withVault([configuration: configuration, vaultSecrets: secrets]) {
      stage("Install Linux Prisma Cloud") {
        sh """
        ansible-playbook -i host --extra-vars "linux_user=${linux_user} linux_pass=${linux_pass}" install_linux_prisma.yaml
        """
      }
    }
  }
}

if (listServices.contains("rollback_linux_prisma")){
  withVault([configuration: configuration, vaultSecrets: secrets]) {
    stage("Rollback Linux Prisma Cloud") {
      sh """
      ansible-playbook -i host --extra-vars "linux_user=${linux_user} linux_pass=${linux_pass}" rollback_linux_prisma.yaml
      """
    }
  }
}

// ------------------ Windows XDR ------------------
if (listServices.contains("install_windows_xdr")){
  withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: "${NEXUS_CRED}", usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASSWORD']]){
    withVault([configuration: configuration, vaultSecrets: secrets]) {
      stage("Install Windows XDR Agent") {
        sh """
        ansible-playbook -i host --extra-vars "win_user=${windows_user} win_pass=${windows_pass} nexus_user=${NEXUS_USER} nexus_password=${NEXUS_PASSWORD}" install_windows_xdr.yaml
        """
      }
    }
  }
}

if (listServices.contains("rollback_windows_xdr")){
  withVault([configuration: configuration, vaultSecrets: secrets]) {
    stage("Rollback Windows XDR") {
      sh """
      ansible-playbook -i host --extra-vars "win_user=${windows_user} win_pass=${windows_pass}" rollback_windows_xdr.yaml
      """
    }
  }
}

// ------------------ Windows Prisma ------------------
if (listServices.contains("install_windows_prisma")){
  withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: "${NEXUS_CRED}", usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASSWORD']]){
    withVault([configuration: configuration, vaultSecrets: secrets]) {
      stage("Install Windows Prisma Cloud") {
        sh """
        ansible-playbook -i host --extra-vars "win_user=${windows_user} win_pass=${windows_pass}" install_windows_prisma.yaml
        """
      }
    }
  }
}

if (listServices.contains("rollback_windows_prisma")){
  withVault([configuration: configuration, vaultSecrets: secrets]) {
    stage("Rollback Windows Prisma Cloud") {
      sh """
      ansible-playbook -i host --extra-vars "win_user=${windows_user} win_pass=${windows_pass}" rollback_windows_prisma.yaml
      """
    }
  }
}
}
*/
// Define helper function outside the node block
/*
def runAnsiblePlaybook(String stageName, String yamlFile, boolean useCredentials, Closure extraVarsClosure, Map configuration, List secrets, String nexusCred) {
    stage(stageName) {
        def step = {
            withVault([configuration: configuration, vaultSecrets: secrets]) {
                sh """
                    ansible-playbook -i host --extra-vars "${extraVarsClosure.call(linux_user, linux_pass, windows_user, windows_pass, env.NEXUS_USER, env.NEXUS_PASSWORD)}" ${yamlFile}
                """
            }
        }
        if (useCredentials) {
            withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: nexusCred, usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASSWORD']]) {
                step()
            }
        } else {
            step()
        }
    }
}
*/
node {
    // Configuration
    def VAULT_ADDR = 'http://172.22.3.91:8200/'
    def VAULT_PATH_SSH = 'linux'
    def NEXUS_CRED = 'nexus_security'
    def NEXUS_ADDR = 'http://172.22.3.92:8081/'

    def secrets = [
        [
            path: "${VAULT_PATH_SSH}", engineVersion: 2,
            secretValues: [
                [vaultKey: 'windows_pass'], [vaultKey: 'windows_user'],
                [vaultKey: 'linux_pass'], [vaultKey: 'linux_user']
            ]
        ]
    ]

    def configuration = [
        vaultUrl: "${VAULT_ADDR}",
        vaultCredentialId: 'd375013e-e3c3-42b8-a417-58b4dee65b99',
        engineVersion: 2
    ]

    // Playbook configurations
    def playbookConfig = [
        'check_connection_linux': [
            stageName: 'Check connection for Linux',
            yamlFile: 'check_connection_linux.yaml',
            credentials: true,
            extraVars: { linux_user, linux_pass, win_user, win_pass, nexus_user, nexus_password ->
                "linux_user=${linux_user} linux_pass=${linux_pass} nexus_user=${nexus_user} nexus_password=${nexus_password}"
            }
        ],
        'check_connection_windows': [
            stageName: 'Check connection for Windows',
            yamlFile: 'check_connection_windows.yaml',
            credentials: true,
            extraVars: { linux_user, linux_pass, win_user, win_pass, nexus_user, nexus_password ->
                "win_user=${win_user} win_pass=${win_pass} nexus_user=${nexus_user} nexus_password=${nexus_password}"
            }
        ],
        'install_linux_xdr': [
            stageName: 'Install Linux XDR Agent',
            yamlFile: 'install_linux_xdr.yaml',
            credentials: true,
            extraVars: { linux_user, linux_pass, win_user, win_pass, nexus_user, nexus_password ->
                "linux_user=${linux_user} linux_pass=${linux_pass} nexus_user=${nexus_user} nexus_password=${nexus_password}"
            }
        ],
        'rollback_linux_xdr': [
            stageName: 'Rollback Linux XDR',
            yamlFile: 'rollback_linux_xdr.yaml',
            credentials: false,
            extraVars: { linux_user, linux_pass, win_user, win_pass, nexus_user, nexus_password -> // Giữ nguyên số lượng params để call không lỗi, dù không dùng hết
                "linux_user=${linux_user} linux_pass=${linux_pass}"
            }
        ],
        'install_linux_prisma': [
            stageName: 'Install Linux Prisma Cloud',
            yamlFile: 'install_linux_prisma.yaml',
            credentials: true,
            extraVars: { linux_user, linux_pass, win_user, win_pass, nexus_user, nexus_password -> // Giữ nguyên số lượng params
                "linux_user=${linux_user} linux_pass=${linux_pass}"
            }
        ],
        'rollback_linux_prisma': [
            stageName: 'Rollback Linux Prisma Cloud',
            yamlFile: 'rollback_linux_prisma.yaml',
            credentials: false,
            extraVars: { linux_user, linux_pass, win_user, win_pass, nexus_user, nexus_password -> // Giữ nguyên số lượng params
                "linux_user=${linux_user} linux_pass=${linux_pass}"
            }
        ],
        'install_windows_xdr': [
            stageName: 'Install Windows XDR Agent',
            yamlFile: 'install_windows_xdr.yaml',
            credentials: true,
            extraVars: { linux_user, linux_pass, win_user, win_pass, nexus_user, nexus_password ->
                "win_user=${win_user} win_pass=${win_pass} nexus_user=${nexus_user} nexus_password=${nexus_password}"
            }
        ],
        'rollback_windows_xdr': [
            stageName: 'Rollback Windows XDR',
            yamlFile: 'rollback_windows_xdr.yaml',
            credentials: false,
            extraVars: { linux_user, linux_pass, win_user, win_pass, nexus_user, nexus_password -> // Giữ nguyên số lượng params
                "win_user=${win_user} win_pass=${win_pass}"
            }
        ],
        'install_windows_prisma': [
            stageName: 'Install Windows Prisma Cloud',
            yamlFile: 'install_windows_prisma.yaml',
            credentials: true,
            extraVars: { linux_user, linux_pass, win_user, win_pass, nexus_user, nexus_password -> // Giữ nguyên số lượng params
                "win_user=${win_user} win_pass=${win_pass}"
            }
        ],
        'rollback_windows_prisma': [
            stageName: 'Rollback Windows Prisma Cloud',
            yamlFile: 'rollback_windows_prisma.yaml',
            credentials: false,
            extraVars: { linux_user, linux_pass, win_user, win_pass, nexus_user, nexus_password -> // Giữ nguyên số lượng params
                "win_user=${win_user} win_pass=${win_pass}"
            }
        ]
    ]

    // Pipeline execution
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
                randomName: 'choice-parameter-5631314439613978', // Giữ nguyên nếu không có lý do thay đổi
                script: [
                    $class: 'GroovyScript',
                    fallbackScript: [classpath: [], sandbox: true, script: 'return[\'Could not get build type\']'],
                    // "Verify" sẽ là default vì nó là item đầu tiên cho PT_SINGLE_SELECT
                    script: [classpath: [], sandbox: true, script: 'return ["Verify", "Install", "Rollback"]']
                ]
            ],
            [
                $class: 'CascadeChoiceParameter',
                choiceType: 'PT_CHECKBOX',
                description: 'Select PlayBook',
                filterLength: 1,
                filterable: true,
                name: 'ansiblePlaybook',
                randomName: 'choice-parameter-banca-5631314456178620', // Giữ nguyên
                referencedParameters: 'build_type',
                script: [ // Script để sinh ra các lựa chọn playbook
                    $class: 'GroovyScript',
                    fallbackScript: [classpath: [], sandbox: true, script: 'return[\'Plz choose something in list\']'],
                    script: [
                        classpath: [], sandbox: true,
                        script: '''
                            if ("Verify".equals(build_type)) {
                                return ['check_connection_linux', 'check_connection_windows']
                            } else if ("Install".equals(build_type)) {
                                return ['install_linux_xdr', 'install_linux_prisma', 'install_windows_xdr', 'install_windows_prisma']
                            } else if ("Rollback".equals(build_type)) {
                                return ['rollback_linux_xdr', 'rollback_linux_prisma', 'rollback_windows_xdr', 'rollback_windows_prisma']
                            }
                            return [] // Trả về danh sách rỗng nếu không khớp
                        '''
                    ]
                ],
                // THÊM defaultValueScript ĐỂ CHỌN MẶC ĐỊNH PLAYBOOK ĐẦU TIÊN
                defaultValueScript: [
                    $class: 'GroovyScript',
                    fallbackScript: [classpath: [], sandbox: true, script: 'return ""'], // Không chọn gì nếu script lỗi
                    script: [
                        classpath: [], sandbox: true,
                        script: '''
                            // build_type sẽ có giá trị mặc định "Verify" khi script này chạy lần đầu
                            if ("Verify".equals(build_type)) {
                                return 'check_connection_linux' // Chọn playbook đầu tiên cho Verify
                            } else if ("Install".equals(build_type)) {
                                return 'install_linux_xdr' // Chọn playbook đầu tiên cho Install
                            } else if ("Rollback".equals(build_type)) {
                                return 'rollback_linux_xdr' // Chọn playbook đầu tiên cho Rollback
                            }
                            return "" // Không chọn gì nếu build_type không khớp
                        '''
                    ]
                ]
            ]
        ])
    ])

    // selectedPlaybooks sẽ lấy giá trị từ params.ansiblePlaybook
    // Nếu defaultValueScript hoạt động, params.ansiblePlaybook sẽ có giá trị mặc định khi build được trigger thủ công lần đầu.
    def selectedPlaybooks = params.ansiblePlaybook.split(',')
    echo "Build Type Selected is: ${params.build_type}"
    echo "Playbooks Selected are: ${selectedPlaybooks}"


    // Run selected playbooks
    selectedPlaybooks.each { playbookName ->
        def trimmedPlaybookName = playbookName.trim() // Trim whitespace phòng trường hợp
        if (playbookConfig.containsKey(trimmedPlaybookName)) {
            def config = playbookConfig[trimmedPlaybookName]
            // Sửa lại cách gọi runAnsiblePlaybook để truyền đúng NEXUS_CRED
            runAnsiblePlaybook(config.stageName, config.yamlFile, config.credentials, config.extraVars, configuration, secrets, NEXUS_CRED)
        } else {
            error "Unknown playbook: ${trimmedPlaybookName}"
        }
    }
}

// Định nghĩa hàm runAnsiblePlaybook (giữ nguyên như trong code của bạn)
def runAnsiblePlaybook(String stageName, String yamlFile, boolean useCredentials, Closure extraVarsClosure, Map configuration, List secrets, String nexusCredId) {
    stage(stageName) {
        def stepLogic = {
            // Truy cập các biến credentials từ Vault (linux_user, linux_pass, windows_user, windows_pass)
            // và Nexus (NEXUS_USER, NEXUS_PASSWORD từ env)
            // Biến linux_user, linux_pass,... này sẽ được unmask bởi withVault
            // Biến NEXUS_USER, NEXUS_PASSWORD này sẽ được unmask bởi withCredentials (nếu useCredentials là true)
            // và được gán vào môi trường (env)
            
            // Thực hiện gọi closure với các giá trị từ Vault và env (Nexus)
            // Các biến linux_user, linux_pass, windows_user, windows_pass sẽ là các biến cục bộ trong scope của withVault
            // Các biến env.NEXUS_USER, env.NEXUS_PASSWORD sẽ là các biến môi trường nếu withCredentials được gọi
            
            def extraVarsString = extraVarsClosure.call(
                binding.getVariables().get('linux_user'), // Lấy từ vault
                binding.getVariables().get('linux_pass'), // Lấy từ vault
                binding.getVariables().get('windows_user'), // Lấy từ vault
                binding.getVariables().get('windows_pass'), // Lấy từ vault
                useCredentials ? env.NEXUS_USER : null, // Lấy từ credentials nếu dùng
                useCredentials ? env.NEXUS_PASSWORD : null // Lấy từ credentials nếu dùng
            )

            sh """
                ansible-playbook -i host --extra-vars "${extraVarsString}" ${yamlFile}
            """
        }

        withVault([configuration: configuration, vaultSecrets: secrets]) {
            if (useCredentials) {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: nexusCredId, usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASSWORD']]) {
                    stepLogic()
                }
            } else {
                // Nếu không dùng credentials, NEXUS_USER và NEXUS_PASSWORD sẽ không được set trong env
                // Closure extraVars cần xử lý trường hợp này (ví dụ: không sử dụng chúng)
                // Hoặc truyền giá trị null/rỗng như đã làm ở trên trong extraVarsClosure.call
                stepLogic()
            }
        }
    }
}
