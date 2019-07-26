---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
---
# Pipeline declarative steps

List of all ALM library [pipeline declarative syntax]((https://jenkins.io/doc/book/pipeline/syntax/)) steps avaliable.

The usual way to invoke an step is alm[step name](option: value, option2: value2...).

For example:

```groovy
//    ...
   stage ('build') {
     steps {
       almGitClone()
       almGradle(task: 'build')
     }
   }
//   ...
```

These declarative methods are base in pipeline scripted classes defined in
[src/com/santander/alm/pipeline] directory, so they are only a way to avoid to include imports
and script sections in a declarative pipeline. You'll always be able to invoke the groovy classes
 directly if you prefer it or if a desired functionality is not implemented as step yet. 

## Azure
Azure tool steps. This steps can only be used in 'azure' type agent.
* [almAzureDeploy](./almAzureDeploy): Deploy (zip or war) in an Azure App Server.

## API Manager
Api Manager steps. This steps can only be used in 'apic' type agent.
* [apic](./apic): Provides a collection of methods for working with API Connect APIs
      
## Git
Git releated steps
* [almGitClone](./almGitClone): Performs a project git clone (checkout scm)
* [almGitHandleMergeRequest](./almGitHandleMergeRequest): Performs a project git clone or a merge (if merge request is detected)
* [almGitUpdateCommitStatus](./almGitUpdateCommitStatus): Updates git commit status with current build status
* [withGitCredentials](./withGitCredentials): Wrapper to execute git commands using configured project credentials
      
## Mail:
* [almMail](./almMail): Sends a notification mail with build result.
        
## Maven:
Maven related steps. These steps can only be used in 'maven' type agent. These method are steps of [declarative Maven](https://github.alm.europe.cloudcenter.corp/ALMOne/pipeline-library/wiki/pipeline#maven) class.
* [almMaven](./almMaven): Executes a maven goal.
* [almMavenGitFlowReleaseEnd](./almMavenGitFlowReleaseEnd): Ends maven gitflow process pushing changes from integration branch to release branch. 
* [almMavenGitFlowReleaseStart](./almMavenGitFlowReleaseStart): Starts maven gitflow process, merging integration and release branch.
* [almMavenGitFlowReleaseWrapper](./almMavenGitFlowReleaseWrapper): Executes a maven git flow release process if  is release var is set. If release, first                                                                            versioning process is executed, then build process indicated in body, and finally commit and push.  
* [almMavenNexusResolver](./almMavenNexusResolver): Executes maven resolver to get uploaded artifact to nexus. 
* [almMavenReadInfoFromPom](./almMavenReadInfoFromPom): Returns a map with pom info.
* [almMavenUpload](./almMavenUpload): Executes maven upload (mvn deploy) to nexus snapshots repository if it version ends with SNAPSHOT otherwise to release repository
* [tibcoCEPipeline](./tibcoCEPipeline): Creates a full maven CI pipeline for Tibco Container Edition.
* [mavenPipeline](./mavenPipeline): Creates a full maven CI pipeline for most common cases.
* [mavenDevOpsPipeline](./mavenDevOpsPipeline): Creates a full maven DevOps pipeline for most common cases.
 
## NPM:
Node Package Manager releated steps. All this steps can only be used in 'node' type agent.
* [almNpmGitFlowReleaseEnd](./almNpmGitFlowReleaseEnd): Ends NPM gitflow process pushing changes from integration branch to release branch. This step, is usually done at the end of the build pipeline if it is a release build.
* [almNpmGitFlowReleaseStart](./almNpmGitFlowReleaseStart): Starts NPM gitflow process,  merging integration and release branch. It also modifies the project version in the package.json file to delete the SNAPSHOT in the release branch and increasing by one the integration SNAPSHOT version. This step, is usually done at the beginning of the build pipeline if it is a release build.
* [almNpmGitFlowReleaseWrapper](./almNpmGitFlowReleaseWrapper): Executes a NPM git flow release process if  is release var is set. If release,
* [npmPipeline](./npmPipeline): Creates a full npm CI pipeline for most common cases.
* [npmDevOpsPipeline](./npmDevOpsPipeline): Creates a full npm DevOps pipeline for most common cases.
firstversioning process is executed, then build process indicated in body, and finally commit and push.

## OpenShift:
OpenShift deploy releated steps. All this steps can only be used in 'ose3-deploy' type agent.
These method are steps of [declarative OpenShift](https://github.alm.europe.cloudcenter.corp/ALMOne/pipeline-library/wiki/pipeline#openshift) class.
* [almOpenshiftDeploy](./almOpenshiftDeploy): Executes a deploy of an OpenShift application into given project.
* [almOpenshiftDeployYaml](./almOpenshiftDeployYaml): (**Deprecated**) Deploys to Openshift using an yaml as configuration.
* [almOpenshiftMultiRegionDeploy](./almOpenshiftMultiRegionDeploy): Deploys to Openshift multiregion with blue-green support.
* [almOpenshiftMultiRegionInput](./almOpenshiftMultiRegionInput): Input approval for deployment that asks for missing deployment credentials.   
* [almOpenshiftSwitch](./almOpenshiftSwitch): Executes a Openshift switch of a deployment executed with the blue-green flag a true.

## Urban Code Deploy:
Urban Code Deploy tool steps. 
* [ucd](./ucd): Provides a collection of methods for working with Urban Code Deploy (need script section).
* [ucdCreateComponentVersion](./ucdCreateComponentVersion): Creates a new component version in UCD.
* [ucdDeployApplication](./ucdDeployApplication): Deploys an application in UCD.
* [ucdGetComponentVersion](./ucdGetComponentVersion): Retrieves all component versions in UCD.

## Code Quality - SONAR
* [almSonar](./almSonar): Executes SonarQube analysis.

## SAS Light - Kiuwan 
* [almKiuwan](./almKiuwan): Executes Kiuwan analysis.

## Utilities:
Utility steps
* [almInput](https://github.alm.europe.cloudcenter.corp/ALMOne/pipeline-library/wiki/almInput): Includes an user input in the pipeline.
* [almInputCredentialsList](https://github.alm.europe.cloudcenter.corp/ALMOne/pipeline-library/wiki//almInputCredentialsList): Includes an user input in the pipeline that shows a list with all current user credentials.
* [almSemanticVersion](https://github.alm.europe.cloudcenter.corp/ALMOne/pipeline-library/wiki//almSemanticVersion): Semantic version methods

* [sshInputAgent](https://github.alm.europe.cloudcenter.corp/ALMOne/pipeline-library/wiki//sshInputAgent): Wraps 'sshagent' pipeline step to allow to use user-scoped credentials of the last input approver 
* [withLastApproverCredentials](https://github.alm.europe.cloudcenter.corp/ALMOne/pipeline-library/wiki//withLastApproverCredentials): Wraps 'withCredentials' pipeline step to allow to use user-scoped credentials of the last input approver
* [almHpAlmJunitUpload](https://github.alm.europe.cloudcenter.corp/ALMOne/pipeline-library/wiki//almHpAlmJunitUpload): Upload in HP_ALM the xUnit test results
