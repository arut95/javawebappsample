import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  stage('init') {
    tool name: 'Maven 3.6.2', type: 'maven'
    sh "printenv | sort"
    checkout scm
  }
  
  stage('build') {
    sh 'mvn clean package'
  }
  
  stage('deploy') {
    def resourceGroup = 'RG-ProdsPlats-Java' 
    def webAppName = 'JavaApp400pm'
    // login Azure
    withCredentials([azureServicePrincipal('AzureAppServiceCredential001')]) {
      sh '''
        az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
        az account set -s $AZURE_SUBSCRIPTION_ID
      '''
    }
    // get publish settings
    def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
    def ftpProfile = getFtpPublishProfile pubProfilesJson
    // upload package
    sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
    // log out
    sh 'az logout'
  }
}
