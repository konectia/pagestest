# [mavenDevOpsPipeline](/vars/mavenDevOpsPipeline.groovy)

Executes a complete DevOps Pipeline pipeline with a release gitflow included for maven projects. _master_ (_'master'_ by default) 
and _integration_ (_'development'_ by default) branchs are required in the project.

This pipeline also needs the 'deployment.yaml' file to define the deployments environments.
 
This _'development'_ pipeline creates following steps:

![Development DevOps Pipeline](./../resources/images/devops-pipeline-development.jpg)

**Stages:**
 * **Build:** Executes _mvn clean verify_ command and archives the junit results (_target/**/TEST-*.xml_)
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

![pipelineParams](/uploads/6dab7396284a0a17c2af0c8d2d1a27ba/pipelineParams.png)




**Parameters:**
* **Global Pipeline parameters**:
    * *masterBranch:* Master branch name (_'master'_ by default)
    * *integrationBranch:* Master branch name (_'development'_ by default)
    * *agent*: Slave build agent (_'maven'_ by default)
    * *deployArtifactId*: Deploy artifact id (by default pom.xml artifact id)
    * *deployArtifactPackaging*: Deploy artifact packaging (by default pom.xml artifact packaging)
    * *deployYmlFile*: Name of the deployment.yaml file to use ('deployment.yaml' by default)
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
 * **ARTIFACT_NEXUS_URL**: Uploaded to Nexus artifact URL
 * **BRANCH_NAME**: Current branch name
 * **GROUP_ID**: Group id
 * **ARTIFACT_ID**: Artifact id
 * **DEPLOY_ARTIFACT_ID:** Deployable artifact id (usually same that ARTIFACT_ID)
 * **DEPLOY_PACKAGING**: Deploy packaging
 * **VERSION**: Pom version

## Example:

Builds a project with a custom command,  executes a Sonar analyisis (_'Serenity'_ instance) and deploys it to
cert, pre and pro environment using the deployment.yaml

```groovy

#!groovy
library 'global-alm-pipeline-library'
devOpsPipeline (
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
        ose3Application: micro-service
        ose3TokenCredentialId: n64168-token-serity-alm-w-dev
        ose3Template: javase
        ose3TemplateParams:
           ARTIFACT_URL: ${ARTIFACT_NEXUS_URL}		
  - name: PRE
    regions:
    - name: PRE-W
      type: ose3
      properties:
        ose3Region: https://api.boaw.paas.gsnetcloud.corp:8443
        ose3Project: serenity-alm-w-pre
        ose3Application: micro-service
        ose3Template: javase
        ose3TemplateParams:
           ARTIFACT_URL: ${ARTIFACT_NEXUS_URL}
  - name: PRO
    regions:
    - name: PRO-W
      type: ose3
      properties:
        ose3Region: https://api.boaw.paas.gsnetcloud.corp:8443
        ose3Project: serenity-alm-w-pro
        ose3Application: micro-service
        ose3Template: javase
        ose3TemplateParams:
           ARTIFACT_URL: ${ARTIFACT_NEXUS_URL}
```
**Note:** This steps creates a full declarative pipeline, so is not possible to add or delete any step