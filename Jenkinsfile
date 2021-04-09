import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=4cc79fba-0463-4f7d-8fb9-b454ef5cc26d',
        'AZURE_TENANT_ID=0adb040b-ca22-4ca6-9447-ab7b049a22ff',
          'AZURE_CLIENT_ID=5c00d252-8711-4cea-94d1-598d282b6a12',
          'AZURE_CLIENT_SECRET=P4jXTr0LMBtTu3rExk5nWL4_3Kp9atPQCf']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'QuickstartJetkins-rg'
      def webAppName = 'newjenkins12-app-omg1'
      // login Azure
      withCredentials([usernamePassword(credentialsId: 'AzureServicePrinciple', passwordVariable: 'P4jXTr0LMBtTu3rExk5nWL4_3Kp9atPQCf', usernameVariable: '5c00d252-8711-4cea-94d1-598d282b6a12')]) {
       sh '''
          az login --testing -u 5c00d252-8711-4cea-94d1-598d282b6a12 -p P4jXTr0LMBtTu3rExk5nWL4_3Kp9atPQCf -t 0adb040b-ca22-4ca6-9447-ab7b049a22ff
          az account set -s 4cc79fba-0463-4f7d-8fb9-b454ef5cc26d
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
}
