# [npmPipeline](/vars/npmPipeline.groovy)

Executes a complete npm pipeline with a release gitflow included. _master_ (_'master'_ by default) 
and _integration_ (_'development'_ by default) branchs are required in the project. 

This pipeline creates following steps:

![Maven Pipeline](./../resources/images/mavenPipeline.jpg)

**Stages:**
 * **Build:** Executes _npm install && npm build_ commands and archives the junit results (_'**/test-results.xml'_)
 * **SonarQube:** SonarQube analysis if sonar instance is defined. If the branch name is the _master branch_ 
 or the _integration branch_  the analysis persist in sonar, else an incremental analysis is executed.
 * **Nexus:** Uploads all build artifact to Nexus repository if branch name is the he _master branch_ (to releases repository)
 or the _integration branch_ (to snapshots repository) and is not a release (because the release branch will do it)
 * **Git Merge (Release):** Commits and pushes the merge between _integration_ and _release_ branches. This
 stage is only executed in release builds.
 * **Deploy:**  If defined, deploys the uploaded artifact to defined OpenShift regions. It is only executed
 if branch name is the _integration_ or _release_ branch and if it is not a relase build.
 * **Post:** Post deploy stage to define deploy test or other post build actions. It is only executed
 if branch name is the _integration_ or _release_ branch and if it is not a relase build.
up
Also adds a _IS_RELEASE_ parameter to create releases from the _integration_ branch.

![Release](./../resources/images/release.png)

**Parameters:**
* **Global Pipeline parameters**:
    * *masterBranch:* Master branch name (_'master'_ by default)
    * *integrationBranch:* Master branch name (_'development'_ by default)
    * *agent*: Slave build agent (_'nodejs'_ by default) or (_'nodejs-v8'_)
    * *applicationGroup*: Application group name (required). Used as a namespace in nexus.
    * *applicationDistDirectory*: Build distribution directory (_dist_ by default).
    This directory will be zipped as build result.
    * *configurationDistDirectory*: Server configuration directory (_conf.d_ by default).
    This directory (if exists) must contain the configuration to apply to the webserver (nginx). 
    * *mail*: Mail options see [almMail](/vars/almMail) ([:] by default)
    * *numToKeepStr*: Number of builds to keep (10 by default)
    * *artifactNumToKeepStr*: Number of artifacts to keep (10 by default)
    * *dir*: If defined assumes that the pom.xml is in that directory
    
* **Build** ('build'):
    * *npmBuildScript*: Build script (_'npm install && npm build'_ by default)
    * *testResults*: Test results to archive (_'**/test-results.xml'_ by default)
    
* **SonarQube** ('sonar'):
    * *sonarInstanceName*: Sonar instance name defined in Jenkins ('Serenity' or 'ISBAN'). No default value. 
    * *sonarProperties*: Map of additional sonar properties.
    * *analisysMode*: Analysis mode (by default preview if is is not the _master_ or the _integration_ branch)
    * *publishHtml*: Publishes the sonar results in Jenkins (true by default)
    
* **Nexus** ('nexus'): No additional parameters

* **Git Merge (Release)** ('merge'): No additional parameters

* **Deploy** ('deploy'):
    * *regions:* Regions to deploy.
    **NOTE**: If you only need to deploy to one region you can write regions properties directly as shown 
    in the examples 
    
* **Post** ('post'):
    * *body*: Code to execute

**Note:** All stages can also define a pre and a post stage code (as a Groovy _Closure_) and agent name.

**Environment variables**:
 * **MASTER_BRANCH:** Master branch name
 * **INTEGRATION_BRANCH**: Integration branch name
 * **APPLICATION_GROUP**: Application group
 * **APPLICATION_NAME**: Application name (project.name in package.json)
 * **APPLICATION_DIST_DIRECTORY**: Distribution directory,
 * **CONFIGURATION_DIST_DIRECTORY**: Configuration distribution directory. 
 * **ARTIFACT_NEXUS_URL**: Uploaded to Nexus artifact URL
 * **CONFIGURATION_ARTIFACT_NEXUS_URL**: Uploaded configuration artifact URL
 * **BRANCH_NAME**: Current branch name
 * **GROUP_ID**: Group id
 * **ARTIFACT_ID**: Artifact id
 * **APPLICATION_VERSION**: package.json version

## Examples:

Default build without sonar and without OpenShift deployment

```groovy
library 'global-alm-pipeline-library'
npmPipeline ()
```

Builds the projectand  executes a Sonar analyisis (Isban instance). 
Avaliable sonar instances are
'ISBAN' and 'Serenity'.

```groovy

library 'global-alm-pipeline-library'
npmPipeline (sonar: [sonarInstanceName: 'ISBAN'])

```

Builds a project with a custom command,  executes a Sonar analyisis (_'Serenity'_ instance) and deploys it to 
_'my-project-dev'_  OpenShift project in _'https://api.boaw.paas.gsnetcloud.corp:8443'_ region using
_'MY-PROJECT-DEPLOY-TOKEN'_ credentials (must be defined in Jenkins as secret text with the deploy token) with
_'my-application'_ name using _'nginx'_ template and giving the _ARTIFACTCONF_URL_ template using the _CONFIGURATION_ARTIFACT_NEXUS_URL_
environment variable (this variable is at Nexus Stage). 


```groovy

library 'global-alm-pipeline-library'
npmPipeline (
    agent: "nodejs-v8", 
    build: [ buildScript: '''
npm install
npm run build -- prod
npm run test -- --browsers PhantomJS --watch=false --code-coverage'''
    ],
    applicationGroup: 'my-group',
    sonar: [ sonarInstanceName: 'Serenity'],
    deploy: [ 
        ose3Region: 'https://api.boaw.paas.gsnetcloud.corp:8443',   // OSE3 region url
        ose3Project: 'my-project-dev',
        ose3TokenCredentialId: 'MY-PROJECT-DEPLOY-TOKEN',
        ose3Application: 'my-application',
        ose3Template: 'nginx',
        ose3TemplateParams: [
            ARTIFACTCONF_URL: '${CONFIGURATION_ARTIFACT_NEXUS_URL}'
        ]
    ])
    
```

**Note:** This steps creates a full declarative pipeline, so is not possible to add or delete any step