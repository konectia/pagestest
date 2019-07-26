# [appianDevOpsPipeline](/vars/appianDevOpsPipeline.groovy)

Executes a complete appian pipeline with a release gitflow included. _master_ (_'master'_ by default) 
and _integration_ (_'development'_ by default) branchs are required in the project. 

This _'development'_ pipeline creates following steps:

![Maven Pipeline](./../resources/images/appianDevopsPipelineDev.jpg)

**Stages:**
 * **Build:** Executes the maven 'clean package' goal.
 * **Nexus:** Uploads all build artifact to Nexus repository to snapshots repository.
 * **Merge:** Commits and pushes the merge between _integration_ and _release_ branches. This stage is only executed in release builds.
 * **Deploy cert:**  Deploys the application to Appian. This stage needs to appianTokenCredentialId to be configured in deployment.yaml file 
 * **Post:** Post deploy stage to define deploy test or other post build actions. 

Also adds a _IS_RELEASE_ parameter to create releases from the _integration_ branch.

![Release](./../resources/images/release.png)


This _'master'_ pipeline creates following steps:

![Maven Pipeline](./../resources/images/appianDevopsPipelineMaster.jpg)

**Stages:**
* **Build**: Executes the maven 'clean package' goal.
* **Nexus**: Uploads all build artifact to Nexus repository to releases repository.
* **Resolve**: Resolve the artifact url from Nexus to deploy in Appian. This stage only runs if the Nexus stage is not executed because the parameter `ONLY_DEPLOY` is true.
* **Deploy pre**: Request the credentials to be used for downloading the properties file and authentication in Appian. Upload the application to Appian by running the Appian client. 
* **Deploy pro**: Request the credentials to be used for downloading the properties file and authentication in Appian. Upload the application to Appian by running the Appian client.
* **Post:** Post deploy stage to define deploy test or other post build actions.
 
Also adds _ONLY_DEPLOY_ parameter in the _master_ branch to deploy without build. Define _DEPLOY_VERSION_ parameter if you dont want to deploy the default version.

![pipelineParams](/uploads/1bb192cb4cb39f0f93a0fd4f5c54e7ab/pipelineParams.png)


**Parameters:**
* **Global Pipeline parameters**:
    * *masterBranch:* Master branch name (_'master'_ by default)
    * *integrationBranch:* Master branch name (_'development'_ by default)
    * *agent*: Slave build agent (_'maven'_ by default)
    * *deployArtifactId*: Deploy artifact id (by default pom.xml artifact id)
    * *deployArtifactPackaging*: Deploy artifact packaging (by default pom.xml artifact packaging)
    * *mail*: Mail options see [almMail](/vars/almMail) ([:] by default)
    * *numToKeepStr*: Number of builds to keep (10 by default)
    * *artifactNumToKeepStr*: Number of artifacts to keep (10 by default)
    * *dir*: If defined assumes that the pom.xml is in that directory
    * *deployYmlFile*: Deployment yaml file to use in deploy stages ('deployment.yaml' as default)
    
* **Build** ('build'):
    * *goal*: Maven goal (_'clean package'_ by default)

* **Nexus** ('nexus'):
    * *arguments*: Maven deploy arguments ('-Dmaven.test.skip=true' by default)
    
* **Merge** ('merge'): No additional parameters

* **Deploy** ('deploy_cert', 'deploy_pre', 'deploy_pro'):
    Uses the configuration defined in the deployment file with the different environments.
    * *environment*: Environment name to look for in the yaml (by default deploy_cert -> cert, deploy_pre -> pre, deploy_pro -> pro)
    * *approve*: If defined an approvation input stage is shown (required for pre and pro environment). Leave empty for any group or add the list of approvers.

_Example of deployment.yaml:_
    
```yaml
environments:
  - name: cert
    appianUrl: http://oclrh70c048.isbcloud.isban.corp:8080/suite/
    appianTokenCredentialId: devAppianCredential
    artifactUrl: ${ARTIFACT_NEXUS_URL}
  - name: pre
    appianUrl: http://oclrh70c048.isbcloud.isban.corp:8080/suite/
    gitUrl: ssh://git@gitlab.alm.gsnetcloud.corp:2220/ALMGLOBAL/appian/example-appian-properties.git
    propertiesPath: pre/example-appian.properties		
    artifactUrl: ${ARTIFACT_NEXUS_URL}
  - name: pro
    appianUrl: http://oclrh70c048.isbcloud.isban.corp:8080/suite/
    gitUrl: ssh://git@gitlab.alm.gsnetcloud.corp:2220/ALMGLOBAL/appian/example-appian-properties.git
    propertiesPath: pro/example-appian.properties			  
    artifactUrl: ${ARTIFACT_NEXUS_URL}
```
Where:
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


_Example of use Jenkinsfile:_

```groovy
appianDevOpsPipeline (
    deploy_cert: [:],
    deploy_pre: [
        environment: 'pre',
        approve: 'impes-developer,impes-technical-lead,impes-product-owner,alm-developer'
    ],
    deploy_pro: [
        environment: 'pro',
        approve: 'impes-developer,impes-technical-lead,impes-product-owner,alm-developer'
    ])
```
