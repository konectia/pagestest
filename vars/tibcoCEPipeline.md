# [tibcoCEPipeline](/vars/tibcoCEPipeline.groovy)

Executes a complete Tibco Container Edition pipeline with a release gitflow included. _master_ (_'master'_ by default) 
and _integration_ (_'development'_ by default) branchs are required in the project.

This pipeline creates following steps:

![Tibco Container Edition Pipeline](./../resources/images/tibcoCEPipeline.jpg)

**Stages:**
 * **Build:** Executes _mvn clean verify_ command and archives the junit results (_target/**/TEST-*.xml_)
 * **SonarQube:** SonarQube analysis if sonar instance is defined. If the branch name is the _master branch_ 
 or the _integration branch_  the analysis persist in sonar, else an incremental analysis is executed.
 * **Nexus:** Uploads all build artifact to Nexus repository if branch name is the he _master branch_ (to releases repository)
 or the _integration branch_ (to snapshots repository) and is not a release (because the release branch will do it)
 * **Git Merge (Release):** Commits and pushes the merge between _integration_ and _release_ branches. This
 stage is only executed in release builds.
 * **Configuration:** If defined, deploys the configuration application
 * **Deploy:**  If defined, deploys the uploaded artifact to defined OpenShift regions. It is only executed
 if branch name is the _integration_ or _release_ branch and if it is not a relase build.
 * **Post:** Post deploy stage to define deploy test or other post build actions. It is only executed
 if branch name is the _integration_ or _release_ branch and if it is not a relase build.

Also adds a _IS_RELEASE_ parameter to create releases from the _integration_ branch and
the _OSE3_DEPLOY_CONFIG_ parameter to indicate if the _Configuration_ stage must be executed or no (usually
after configuration service deployment it is not required to redeploy it)	



![Release](./../resources/images/tibcoCE-release.png)

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
    * *dir*: If defined assumes that the pom.xml is in that directory.
    * *deployConfig*: _OSE3_DEPLOY_CONFIG_ parameter default value (true if not defined)
    
* **Build** ('build'):
    * *goal*: Maven goal (_'clean verify'_ by default)
    * *testResults*: Test results to archive (_''target/**/TEST-*.xml'_ by default)
    
* **SonarQube** ('sonar'):
    * *sonarInstanceName*: Sonar instance name defined in Jenkins ('Serenity' or 'ISBAN'). No default value. 
    * *sonarProperties*: Map of additional sonar properties.
    * *analisysMode*: Analysis mode (by default preview if is is not the _master_ or the _integration_ branch)
    * *publishHtml*: Publishes the sonar results in Jenkins (true by default)
    
* **Nexus** ('nexus'): No additional parameters

* **Git Merge (Release)** ('merge'): No additional parameters
* **Configuration** ('configuration'): See Deploy
* **Deploy** ('deploy'):
    * *regions:* Regions to deploy.
    **NOTE**: If you only need to deploy to one region you can write regions properties directly as shown 
    in the examples 
    
* **Post** ('post'):
    * *body*: Code to execute

**Important:** This pipeline assumes that the maven parent project is in a _*.parent folder
and the deployable artifact is called with the same name that the *.parent folder without _.parent_ suffix.
If this is not the case provide the _dir_ parameter with the parent folder
and the _deployArtifactId_ parameter with the deployable artifact id.

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
tibcoCEPipeline ()
```

Builds the application, executes a SonarQube analysis (_ISBAN_ instance) and deploys it on OpenShift. 

```groovy

library 'global-alm-pipeline-library'
tibcoCEPipeline (sonar: [sonarInstanceName: 'ISBAN'],
    deploy: [ 
        ose3Region: 'https://api.boaw.paas.gsnetcloud.corp:8443',
        ose3Project: 'my-project-dev',
        ose3TokenCredentialId: 'MY-PROJECT-DEPLOY-TOKEN',
        ose3Application: 'mytibcoapp',
        ose3Template: 'bwce',
        ose3TemplateParams: [
            BW_PROFILE: 'Docker'                 // BW profile
        ]
    ])

```

A more complete example that defines the _master_ and _integration_ branch names, the agent and add a post
deploy action that saves the application endpoint to be used in the post stage. It also deploys a configuration
service. 

A post _'Post'_ stage is defined to execute some test to deployed application using _'newman'_.

```groovy
library 'global-alm-pipeline-library'
// Configuration Service application name
// This name is used to deploy or update de configuration Service
// and is used also as parameter in the SPRING_CLOUD_CONFIG_SERVER_URL application parameter
final OSE3_DEPLOY_CONFIG_APP_NAME = 'tibco-app-config'
// git URL with the configuration for the configuration service
// if you don't need a configuration service or you don't need to
// deploy the configuration service this variable is not required
final CONFIGURATION_GIT_REPOSITORY_URL = 'https://gitlab.alm.gsnetcloud.corp/sebpm/config-repo.git'
// OpenShift deploy configuration
// change this according to your region (boe, boaw,...)
// project and credentialID
final OSE3_DEPLOY_REGION = 'https://api.boaw.paas.gsnetcloud.corp:8443'
// OpenShift project name
final OSE3_DEPLOY_PROJECT = 'my-project-dev'
// This credential must exist in your jenkins instance
// as a Secret text global credential	
final OSE3_DEPLOY_CREDENTIAL_ID =  'MY-PROJECT-DEPLOY-TOKEN'

tibcoCEPipeline (
    masterBranch: 'master',
    integrationBranch: 'development',
    agent: 'maven',
    sonar: [sonarInstanceName: 'ISBAN'],
    configuration: [ 
        ose3Region: OSE3_DEPLOY_REGION,
        ose3Project: OSE3_DEPLOY_PROJECT,
        ose3TokenCredentialId: OSE3_DEPLOY_CREDENTIAL_ID,
        ose3Application: OSE3_DEPLOY_CONFIG_APP_NAME,
        ose3Template: 'config-service-git',
        ose3TemplateParams: [
            SPRING_CLOUD_CONFIG_SERVER_GIT_URI: CONFIGURATION_GIT_REPOSITORY_URL
        ]
    ],
    deploy: [ 
        ose3Region: OSE3_DEPLOY_REGION,
        ose3Project: OSE3_DEPLOY_PROJECT,
        ose3TokenCredentialId: OSE3_DEPLOY_CREDENTIAL_ID,
        ose3Application: 'tibcoce',
        ose3Template: 'bwce',
        ose3TemplateParams: [
            BW_PROFILE: 'Docker',                 // BW profile
            APP_CONFIG_PROFILE: 'DEV',            // Profile inthe configuration Service
            SPRING_CLOUD_CONFIG_SERVER_URL: "http://${OSE3_DEPLOY_CONFIG_APP_NAME}:8080/" // configuration service URL
        ],  
        post: { pipeline ->
            echo "Getting ROUTE URL..."
            def prop = readProperties  file: 'deploy_jenkins.properties'
            env.OSE3_END_POINT_URL = prop['OSE3_END_POINT_URL']
            // stores the ose3 route endpoint to use it in tests
            echo "OSE3_END_POINT_URL = ${OSE3_END_POINT_URL}"
        }
    ],
    post: [
        agent: 'nodejs', 
        body: { pipeline -> 
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