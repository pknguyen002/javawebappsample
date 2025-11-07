import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=fc3a05c2-95b2-4838-90b3-3921132cce78',
         'AZURE_TENANT_ID=c48f589c-6750-4f2a-ac8c-cfe675eb9d81'])
 {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
    def resourceGroup = 'jenkins-get-started-rg'
    def webAppName = 'kimjava31647b'

    // login Azure
    withCredentials([azureServicePrincipal('AzureServicePrincipal')]) {
        sh '''
        az cloud set --name AzureCloud
        az account clear
        az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID
        az account set --subscription $AZURE_SUBSCRIPTION_ID
        '''
    }

    // get publish settings
    def pubProfilesJson = sh(
        script: "az webapp deployment list-publishing-profiles -g ${resourceGroup} -n ${webAppName}",
        returnStdout: true
    ).trim()

    def ftpProfile = getFtpPublishProfile(pubProfilesJson)

    // upload package
    sh "curl -T target/calculator-1.0.war ${ftpProfile.url}/webapps/ROOT.war -u '${ftpProfile.username}:${ftpProfile.password}'"

    // log out
    sh 'az logout'
}

}
}
