# [npmDevOpsPipeline](/vars/npmDevOpsPipeline.groovy)

Executes a complete DevOps Pipeline npm pipeline with a release gitflow included. _master_ (_'master'_ by default) 
and _integration_ (_'development'_ by default) branchs are required in the project.

This pipeline also needs the 'deployment.yaml' file to define the deployments environments.
 
This _'development'_ pipeline creates following steps:

![Development DevOps Pipeline](./../resources/images/devops-pipeline-development.jpg)

**Stages:**
 * **Build:** Executes the npm install && npm build command
 * **Sonar:** Executes the SonarQube analysis
 * **Nexus:** Uploads the contents of 'dist' dist directory to Nexus repositor to snapshots repository as a zip file.
 * **Merge:** Commits and pushes the merge between _integration_ and _release_ branches. This stage is only executed in release builds.
 * **Deploy cert:**  Deploys the artifact uploaded to Nexus. 
 * **Test cert:** Post deploy stage to define deploy test or other post build actions. 

Also adds a _IS_RELEASE_ parameter to create releases from the _integration_ branch.

![Release](./../resources/images/release.png)


This _'master'_ pipeline creates following steps:

![Master_DevOps pipeline](./../resources/images/devops-pipeline-master.jpg)

**Stages:**
* **Build:** Executes the npm install && npm build command
* **Sonar:** Executes the SonarQube analysis
* **Nexus**: Uploads all build artifact to Nexus repository to releases repository.
* **Resolve**: Resolve the artifact url from Nexus to deploy. This stage only runs if the Nexus stage is not executed because the parameter `ONLY_DEPLOY` is true.
* **Deploy pre**: Deploys the application in pre-production environment. If appoval is required the the deployment is stopped until a user aproves and gives required deployment credentials.
* **Test pre**:  Preproduction test stage. 
* **Deploy pro**: Deploys the application in production environment. If appoval is required the the deployment is stopped until a user aproves and gives required deployment credentials.
 

Also adds _ONLY_DEPLOY_ parameter in the _master_ branch to deploy without build. Define _DEPLOY_VERSION_ parameter if you dont want to deploy the default version.

![pipelineParams](/uploads/c8f6fed5f61fdf2d0cf7690c6589ee07/pipelineParams.png)




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
    * *deployYmlFile*: Name of the deployment.yaml file to use ('deployment.yaml' by default) 
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

* **Deploy cert / Deploy pre / Deploy pro** ('deploy_cert', 'deploy_pre', 'deploy_pro'):
    * environment: Environment name in deployment.yaml (by default it uses 'cert', 'pre' and 'pro')
    * approve: If present, the deploy must be confirm, so the pipeline stops until an user approves it. This parameter
    can be with the empty value (any user can approve the pipeline or only the groups specified). If not 
    present, deployment credential must be specified in the deployment yaml.
    
**NOTE:** if a deploy stage is not present in the Jenkinsfile. The deploy is skipped.    
* **Test cert / Test pre** ('post'):
    * *body*: Test code to execute (optional)

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

## Example:

Builds a project with a custom command,  executes a Sonar analyisis (_'Serenity'_ instance) and deploys it to
cert, pre and pro environment using the deployment.yaml

```groovy

#!groovy
library 'global-alm-pipeline-library'
npmDevOpsPipeline (
	applicationGroup: 'mi.npm.test',
	build: [
	    buildScript: '''
set -e
npm install
npm run build -- --prod
npm run test -- --browsers PhantomJS --watch=false --code-coverage
''',
        testResults: 'junit/test-result.xml'
	],
    sonar: [ sonarInstanceName: 'Serenity'],
    deploy_cert: [
	    environment: 'CERT'
	],
	deploy_pre: [
	    environment: 'PRE'
	],
	deploy_pro: [
	    environment: 'PRO',
	    approve: 'alm-impes'
	]
)

```


deployment.yaml
```yaml
environments:
  - name: CERT
    regions:
    - name: CERT-W
      type: ose3
      properties:
        ose3Region: https://api.boaw.paas.gsnetcloud.corp:8443
        ose3Project: serenity-alm-w-dev
        ose3Application: samcglobal
        ose3TokenCredentialId: n64168-token-serity-alm-w-dev
        ose3Template: nginx
        ose3TemplateParams:
           ARTIFACT_URL: ${ARTIFACT_NEXUS_URL}
           ARTIFACTCONF_URL: ${CONFIGURATION_ARTIFACT_NEXUS_URL}		
  - name: PRE
    regions:
    - name: PRE-W
      type: ose3
      properties:
        ose3Region: https://api.boaw.paas.gsnetcloud.corp:8443
        ose3Project: serenity-alm-w-pre
        ose3Application: samcglobal
        ose3Template: nginx
        ose3TemplateParams:
           ARTIFACT_URL: ${ARTIFACT_NEXUS_URL}
           ARTIFACTCONF_URL: ${CONFIGURATION_ARTIFACT_NEXUS_URL}
  - name: PRO
    regions:
    - name: PRO-W
      type: ose3
      properties:
        ose3Region: https://api.boaw.paas.gsnetcloud.corp:8443
        ose3Project: serenity-alm-w-pro
        ose3Application: samcglobal
        ose3Template: nginx
        ose3TemplateParams:
           ARTIFACT_URL: ${ARTIFACT_NEXUS_URL}
           ARTIFACTCONF_URL: ${CONFIGURATION_ARTIFACT_NEXUS_URL}
```
**Note:** This steps creates a full declarative pipeline, so is not possible to add or delete any step