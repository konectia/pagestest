# [almOpenshiftDeployYaml](/vars/almOpenshiftDeployYaml.groovy)
**Deprecated:** This step has been deprecated in favor of [almOpenshiftMultiRegionDeploy](almOpenshiftMultiRegionDeploy)
because for multiregion deployment different parameters by region may be required (i.e. credentialId)

Deploy to Openshift using a Yaml as configuration.

**Parameters**
 * *configurationFilePath*: Path to the Yaml file. The path **must** be relative to git project directory.
 * *environment*: Name of the environment configuration to use.
 * *params*: Map of values to be interpolated in the Yaml configuration (Optional).
 * *credId*: Credential to override the specified in the Yaml configuration  (Optional).

```groovy
stage ('Deploy') {
  agent {
    label 'ose3-deploy'
  }
  steps {
    almOpenshiftDeployYaml(
        configurationFilePath: "application.yml",
        environment: "development",
        params: [ose3TemplateParams: [ARTIFACT_URL: artifactUrl]])
  }
}
```

The Yaml file must have the following structure:

```yml
 deploy:
   environments:
     - name: [Environment name]
       regions:
         - name: [Region name]
           type: ose3
           properties:
             ose3Region: [Region URL]
             ose3Project: [Project]
             ose3TokenCredentialId: [Deploy credential id]
             ose3Application: [Application name]
             ose3Template: [Template]
             ose3TemplateParams:
               ARTIFACT_URL: [Artifact url]
             ose3TemplateSecretsParams:
             ose3BlueGreen: [Boolean] (Optional)
             ose3Rollback: [Boolean] (Optional)
 ```

The Yaml file can have more than one environment declared. The parameter `environment` allows selecting the properties to be used.

The configuration values of the elements below the `ose3TemplateParams` and `ose3Application` elements of the Yaml file can be customized leveraging variable interpolation (*i.e.*, `${var_name}`). The values to be interepolated will be taken from a `Map` passed using the `param` argument.

The `ose3TokenCredentialId` configuration value may also be overriden via the parameter `credId`.

**NOTE:** This step must be executed in an *ose3-deploy* agent.
