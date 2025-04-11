def NEXUS_CRED = 'nexus_security'


withCredentials([[$class: 'UsernamePasswordMultiBinding', 
  credentialsId: "${NEXUS_CRED}", 
  usernameVariable: 'NEXUS_USER', 
  passwordVariable: 'NEXUS_PASSWORD']]) {
    sh '''
      echo "Username: $NEXUS_USER"
      echo "Password Length: ${#NEXUS_PASSWORD}"
    '''
}
