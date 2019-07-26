# [qlikviewPipeline](/vars/qlikviewPipeline.groovy)

Executes a complete qlikview pipeline with a release gitflow included. _master_ (_'master'_ by default) 
and _integration_ (_'development'_ by default) branchs are required in the project. 

This pipeline creates following steps:

![CI Pipeline](./../resources/images/ci-pipeline.jpg)

**Stages:**
 * **Build:** Executes _mvn clean verify_ command and archives the junit results (_target/**/TEST-*.xml_)
 * **SonarQube:** SonarQube analysis if sonar instance is defined. If the branch name is the _master branch_ 
 or the _integration branch_  the analysis persist in sonar, else an incremental analysis is executed.
 * **Nexus:** Uploads all build artifact to Nexus repository if branch name is the he _master branch_ (to releases repository)
 or the _integration branch_ (to snapshots repository) and is not a release (because the release branch will do it)
 * **Git Merge (Release):** Commits and pushes the merge between _integration_ and _release_ branches. This
 stage is only executed in release builds.
 * **Deploy:**  If not defined, by default it deploys the uploaded artifact to defined UrbanCode. If defined, you can define the type of deployment to be made: urbancode, ansible, ... It is only executed if branch name is the _integration_ or _release_ branch and if it is not a relase build.
 * **Post:** Post deploy stage to define deploy test or other post build actions. It is only executed
 if branch name is the _integration_ or _release_ branch and if it is not a relase build.

Also adds a _IS_RELEASE_ parameter to create releases from the _integration_ branch.

![Release](./../resources/images/release.png)

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
    
* **Build** ('build'):
    * *goal*: Maven goal (_'clean verify'_ by default)
    * *testResults*: Test results to archive (_''target/**/TEST-*.xml'_ by default)
    
* **SonarQube** ('sonar'):
    * *sonarInstanceName*: Sonar instance name defined in Jenkins ('Serenity' or 'ISBAN'). No default value. 
    * *sonarProperties*: Map of additional sonar properties.
    * *analisysMode*: Analysis mode (by default preview if is is not the _master_ or the _integration_ branch)
    * *publishHtml*: Publishes the sonar results in Jenkins (true by default)
    
* **Nexus** ('nexus'):
    * *arguments*: Maven deploy arguments ('-Dmaven.test.skip=true' by default)
* **Git Merge (Release)** ('merge'): No additional parameters

* **Deploy** ('deploy'):
    * type: Parameter located in Jenkinsfile or deployment.yaml that indicated the deploy type (urbancode, ansible, ...). If not defined, the type by default is urbancode.
        
* **Post** ('post'):
    * *body*: Code to execute

**Note:** All stages can also define a pre and a post stage code (as a Groovy _Closure_) and agent name.

**Environment variables**:
 * **MASTER_BRANCH:** Master branch name
 * **INTEGRATION_BRANCH**: Integration branch name
 * **ARTIFACT_NEXUS_URL**: Uploaded to Nexus artifact URL
 * **BRANCH_NAME**: Current branch name
 * **GROUP_ID**: Group id
 * **ARTIFACT_ID**: Artifact id
 * **DEPLOY_ARTIFACT_ID:** Deployable artifact id (usually same that ARTIFACT_ID)
 * **DEPLOY_PACKAGING**: Deploy packaging
 * **VERSION**: Pom version

## Examples:

Default build without sonar and without OpenShift deployment

```groovy
library 'global-alm-pipeline-library'
qlikviewPipeline ()
```

Build of java library that executes a Sonar analyisis (Isban instance). 
Avaliable sonar instances are
'ISBAN' and 'Serenity'.

```groovy

library 'global-alm-pipeline-library'
qlikviewPipeline (sonar: [sonarInstanceName: 'ISBAN'])

```

Build of java library that executes a Sonar analyisis (_'Serenity'_ instance) and deploys it to 
  - _server:_ (Required) UCD server URL (i.e. 'https://ucd-server.santander.corp:8443')
  - _userCredentials:_ (Required) UCD connection user name credential
  - _application:_ (Required) UCD Application name
  - _environment:_ (Required) UCD Deployment environment
  - _process:_ (Required) UCD application process

Optional variables:

 * _component_:   Component name  (i.e. 'com.santander.alm.,) ? If it is not indicated, it will be built with: _GROUP_ID:ARTIFACT_ID_
 * _version_: Version name to create (i.e. '1.0.0')  ?  If it is not indicated, it will be built  with the variable _VERSION_
 * _versionProperties_:  map of version properties (i.e. [ARTIFACT_URL: 'https://nexus.alm.com/maven-releases/myartifact.zip'])
                  ARTIFACT_URL:
                  
```groovy

library 'global-alm-pipeline-library'
qlikviewPipeline (sonar: [sonarInstanceName: 'Serenity'],
    deploy: [
      type: 'urbancode',
      userCredentials: <urbancode-credential>,
      server: <urbancode-url>,
      application: <application-name>,
      environment: <urbancode-environment>,
      process: deployApplication,
      component: <name-component>,
      version: <component-version>,
      versionProperties: [
        ARTIFACT_URL: '${ARTIFACT_NEXUS_URL}'  
      ]
    ],)
    
```
Deploy with ansible
* _type_  Type of deploy (ose3, ose3Yaml, ansible)
* _ansibleCredentialId_ Credentials ssh to conect to linux machines (must be defined in Jenkins as SSH username with private key)
* _winrmCredentialId_  Credentials to conect to windows machines (must be defined in Jenkins as Username with password)
* _inventory_  File contains the hosts list to deploy the zip file (optional, by default 'hosts')
* _playbook_  Name of playbook to execute (optional, by default 'playbook.yml')
* _ansibleGalaxy_  Boolean indicates if download ansible roles (optional, by default false)
* _requirements_  File configures ansible roles to download (optional, by default 'requirements.yml')
* _download_url_  Nexus url of zip file to deploy (optional)
* _sourceFile_  Path, inside of project, of zip file to deploy (optional)

**Note:** At least one of the two attributes, ansibleCredentialId or winrmCredentialId, must be defined. 
At least one of the two attributes, download_url or sourceFile, must be defined")


```groovy

library 'global-alm-pipeline-library'
qlikviewPipeline (sonar: [sonarInstanceName: 'Serenity'],
  deploy: [
      type: 'ansible',
      ansibleCredentialId: 'CREDENTIAL-SSH',
      winrmCredentialId: 'WINRM-CREDENTIAL-USER-PASSWORD',
      inventory: 'environments/hosts',
      playbook: 'main.yml',
      ansibleGalaxy: true,
      requirements: '',
      extraParams: [
  	    download_url: '${ARTIFACT_NEXUS_URL}',
        sourceFile: 'pathFileToDeployInProject'
      ],
)

```


**Note:** This steps creates a full declarative pipeline, so is not possible to add or delete any step