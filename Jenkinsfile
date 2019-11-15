import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  stage('init') {
    mvnHome = tool 'MVN3'
    checkout scm
  }

  stage('build') {
        //Use MVN_Home to locate Maven
    withEnv(["MVN_HOME=$mvnHome"]) {
         if (isUnix()) {
            sh '"$MVN_HOME/bin/mvn" clean package'
         } else {
            bat(/"%MVN_HOME%\bin\mvn" clean package/)
         }
      }
  }
  
  stage('deploy') {
    def resourceGroup = 'RG-ProdsPlats-Java' 
    def webAppName = 'JavaApp400pm'
    // login Azure
    withCredentials([azureServicePrincipal('AzureAppServiceCred4Java001')]) {
      sh '''
        /usr/local/bin/az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
        /usr/local/bin/az account set -s $AZURE_SUBSCRIPTION_ID
      '''
    }
    // get publish settings
    def pubProfilesJson = sh script: "/usr/local/bin/az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
    def ftpProfile = getFtpPublishProfile pubProfilesJson
    // upload package
    sh "curl -T target/calculator-1.0.war $ftpProfile.url/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
    // log out
    sh '/usr/local/bin/az logout'
  }
}
