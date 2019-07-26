# [almAppianDeploy](/vars/almAppianDeploy.groovy)

Executes a deploy of an Appian application into given project.

**Parameters:**
- *artifactUrl:* Artifact url to deploy in Appian
- *appianDeploymentUserName:* Appian user name to conect Appian. If a credential is provided, the user name will be obtained using `withCredentials`. If there was previously an approval, the user name will be obtained from the approval.
- *appianDeploymentPassword:* Appian password to conect Appian. If a credential is provided, the password will be obtained using `withCredentials`. If there was previously an approval, the password will be obtained from the approval.
- *gitCredentialId:* Git credential id to download the properties file 
- *propertiesGitUrl:* Git project url that saves the properties files from each environment 
- *propertiesPath:* Properties path into git project
- *appianAdmImportClientJar:* Nexus url to download the client jar that deploys the application in Appian
- *appianAdmImportClientProperties:* Nexus url to download the client properties file that is necessary to execute the client
- *appianUrl:* Appian url where to deploy


**NOTE:** If the properties file is saved into appian application, to fill with null the parameters: gitCredentialId, propertiesGitUrl and propertiesPath.

```groovy

stage("Deploy") {
	environment {
		APPIAN_URL = 'http://oclrh70c048.isbcloud.isban.corp:8080/suite/'
		APPIAN_CREDENTIAL = 'devAppianCredential'
	}			
	steps {
		withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: env.APPIAN_CREDENTIAL,
			usernameVariable: 'APPIAN_DEPLOYMENT_USERNAME', passwordVariable: 'APPIAN_DEPLOYMENT_PASSWORD']]){
			almAppianDeploy ([artifactUrl: artifactUrl, 
				appianDeploymentUserName: env.APPIAN_DEPLOYMENT_USERNAME,
				appianDeploymentPassword: env.APPIAN_DEPLOYMENT_PASSWORD, 
				gitCredentialId: null, 
				propertiesGitUrl: null, 
				propertiesPath: null, 
				appianAdmImportClientJar: env.APPIAN_ADM_IMPORT_CLIENT_JAR,
				appianAdmImportClientProperties: env.APPIAN_ADM_IMPORT_CLIENT_PROPERTIES,
				appianUrl: env.APPIAN_URL])		
		}					
	}
}
	
```

```groovy

stage('Deploy') {
	environment {
		APPIAN_URL = 'http://oclrh70c048.isbcloud.isban.corp:8080/suite/'
		GIT_URL = 'ssh://git@gitlab.alm.gsnetcloud.corp:2220/ALMGLOBAL/appian/example-appian-properties.git'
	}				
	steps {				
		script {
			propertiesPath = "pre/${pomMap['POM_ARTIFACTID']}.properties"		
		}

		almAppianDeploy ([artifactUrl: artifactUrl, 
			appianDeploymentUserName: userInput.Username,
			appianDeploymentPassword: userInput.Password, 
			gitCredentialId: userInput.GitCredentialId, 
			propertiesGitUrl: env.GIT_URL, 
			propertiesPath: propertiesPath, 
			appianAdmImportClientJar: env.APPIAN_ADM_IMPORT_CLIENT_JAR,
			appianAdmImportClientProperties: env.APPIAN_ADM_IMPORT_CLIENT_PROPERTIES,
			appianUrl: env.APPIAN_URL])				
	}
}

```
