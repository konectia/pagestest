# [mavenPipeline](/vars/mavenPipeline.groovy)

Executes a complete maven pipeline with a release gitflow included. _master_ (_'master'_ by default) 
and _integration_ (_'development'_ by default) branchs are required in the project. 

This pipeline creates following steps:

![Maven Pipeline](./../resources/images/mavenPipeline.jpg)

**Stages:**
 * **Build:** Executes _mvn clean verify_ command and archives the junit results (_target/**/TEST-*.xml_)
 * **SonarQube:** SonarQube analysis if sonar instance is defined. If the branch name is the _master branch_ 
 or the _integration branch_  the analysis persist in sonar, else an incremental analysis is executed.
 * **Kiuwan:** SAS Light (Kiuwan) analysis.
 * **Nexus:** Uploads all build artifact to Nexus repository if branch name is the he _master branch_ (to releases repository)
 or the _integration branch_ (to snapshots repository) and is not a release (because the release branch will do it)
 * **Git Merge (Release):** Commits and pushes the merge between _integration_ and _release_ branches. This
 stage is only executed in release builds.
 * **Deploy:**  If not defined, by default it deploys the uploaded artifact to defined OpenShift. If defined, you can define the type of deployment to be made: OpenShift, Ansible, ... It is only executed if branch name is the _integration_ or _release_ branch and if it is not a relase build.
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

* **Kiuwan** ('kiuwan'):
    * *credentialsId [optional]*: credentialsId of the user that execute Kiuwan. By default the pipeline use System Kiuwan User.
    * *kiuwanProperties [optional]:* Map of additional kiuwan properties.

* **Nexus** ('nexus'):
    * *arguments*: Maven deploy arguments ('-Dmaven.test.skip=true' by default)

* **Git Merge (Release)** ('merge'): No additional parameters

* **Deploy** ('deploy'):
    * *type:* Type of deploy: ose3 (Openshift), ose3Yaml (Openshift with yaml deployment file), ansible ([Ansible](/vars/ansible,... If not defined, the type by default is Openshift.
    * *regions:* Regions to deploy, only to Openshift.
    **NOTE**: If you only need to deploy in Openshift to one region you can write regions properties directly as shown 
    in the examples 
    
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
mavenPipeline ()
```

Build of java library that executes a Sonar analyisis (Isban instance). 
Avaliable sonar instances are
'ISBAN' and 'Serenity'.

```groovy

library 'global-alm-pipeline-library'
mavenPipeline (sonar: [sonarInstanceName: 'ISBAN'])

```

Build of java library that executes a Sonar analyisis (_'Serenity'_ instance) and deploys it to 
_'my-project-dev'_  OpenShift project in _'https://api.boaw.paas.gsnetcloud.corp:8443'_ region using
_'MY-PROJECT-DEPLOY-TOKEN'_ credentials (must be defined in Jenkins as secret text with the deploy token) with
_'my-application'_name using _'javase'_ template. 


```groovy

library 'global-alm-pipeline-library'
mavenPipeline (sonar: [sonarInstanceName: 'ISBAN'],
    deploy: [ 
        ose3Region: 'https://api.boaw.paas.gsnetcloud.corp:8443',
        ose3Project: 'my-project-dev',
        ose3TokenCredentialId: 'MY-PROJECT-DEPLOY-TOKEN',
        ose3Application: 'my-application',
        ose3Template: 'javase'
    ])
    
```
Deploy with ansible
* _'type'_  Type of deploy (ose3, ose3Yaml, ansible)
* _'ansibleCredentialId'_ Credentials ssh to conect to linux machines (must be defined in Jenkins as SSH username with private key)
* _'winrmCredentialId'_  Credentials to conect to windows machines (must be defined in Jenkins as Username with password)
* _'inventory'_  File contains the hosts list to deploy the zip file (optional, by default 'hosts')
* _'playbook'_  Name of playbook to execute (optional, by default 'playbook.yml')
* _'ansibleGalaxy'_  Boolean indicates if download ansible roles (optional, by default false)
* _'requirements'_  File configures ansible roles to download (optional, by default 'requirements.yml')
* _'download_url'_  Nexus url of zip file to deploy (optional)
* _'sourceFile'_  Path, inside of project, of zip file to deploy (optional)

**Note:** At least one of the two attributes, ansibleCredentialId or winrmCredentialId, must be defined. 
At least one of the two attributes, download_url or sourceFile, must be defined")


```groovy

library 'global-alm-pipeline-library'
mavenPipeline (sonar: [sonarInstanceName: 'Serenity'],
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

A more complete example that defines the _master_ and _integration_ branch names, the agent and add a post
deploy action that saves the application endpoint to be used in the post stage.

NOTE: In this case the deploy uses the multi-region deploy format

It also uses the _'Post'_ stage to execute some test to deployed application using _'newman'_.

See [deployment.yaml](/deployment_yaml) for more information.
 
```groovy

library 'global-alm-pipeline-library'
mavenPipeline (
    masterBranch: 'master',
    integrationBranch: 'develop',
    agent: 'maven',
    sonar: [sonarInstanceName: 'ISBAN'],
    kiuwan: [],
    deploy: [ 
        regions: [
            [
            name: 'dev',
            type: 'ose3',
            properties: [
                ose3Region: 'https://api.boaw.paas.gsnetcloud.corp:8443',
                ose3Project: 'my-project-dev',
                ose3TokenCredentialId: 'MY-PROJECT-DEPLOY-TOKEN',
                ose3Application: 'my-application',
                ose3Template: 'javase',
                ose3TemplateParams: [
                    JAVA_OPTS_EXT: '-DmyExtraParam=${BUILD_ID}'
                    ]
                ]
            ]
        ],
        post: { 
            echo "Getting ROUTE URL..."
            def prop = readProperties  file: 'deploy_jenkins.properties'
            env.OSE3_END_POINT_URL = prop['OSE3_END_POINT_URL']
            // stores the ose3 route endpoint to use it in tests
            echo "OSE3_END_POINT_URL = ${OSE3_END_POINT_URL}"
        }
    ],
    post: [
        agent: 'nodejs', 
        body: {
            try{
                echo "Executing acceptance tests..."
                sh ('newman run acceptance/test-ms.postman.collection.json --global-var ROUTE_URL=${OSE3_END_POINT_URL} --global-var POM_VERSION=${VERSION} --insecure -r html,cli,junit --reporter-html-export newman/index.html')
            }finally {
                // archives junit format test results
                junit allowEmptyResults: true, testResults: 'newman/*.xml'
                // archives the test report
                publishHTML([allowMissing: true, alwaysLinkToLastBuild: false, includes: '**/*.html', keepAll: false, reportDir: 'newman', reportFiles: 'index.html', reportName: 'Newman Acceptance Test Report', reportTitles: ''])
            }
        }
    ])
    
```

**Note:** This steps creates a full declarative pipeline, so is not possible to add or delete any step