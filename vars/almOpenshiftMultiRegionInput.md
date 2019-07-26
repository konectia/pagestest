# [almOpenshiftMultiRegionInput](/vars/almOpenshiftMultiRegionInput.groovy)

Approval input that waits for deployment approval with multi-region support.
If Openshift credentials are not provided for a region it asks for them.
This step, returns the given information provided by the submitter.

**Parameters:**
- *yaml:* Deployment configuration to parse it can be read from a yaml with  [readYaml](https://jenkins.io/doc/pipeline/steps/pipeline-utility-steps/#code-readyaml-code-read-yaml-from-files-in-the-workspace-or-text) step.

```yaml
deploy:
    environments:
      - name: PRE
        regions:
        - name: PRE-EAST
          type: ose3
          properties:
            ose3Region: https://api.boae.paas.gsnetcloud.corp:8443
            ose3Project: myProject-pre
            ose3Application: myApp
            ose3TokenCredentialId:
            ose3Template: javase
            ose3TemplateParams:
              ARTIFACT_URL: ${ARTIFACT_URL}
        - name: PRE-WEST
          type: ose3
          properties:
            ose3Region: https://api.boaw.paas.gsnetcloud.corp:8443
            ose3Project: myProject-pre
            ose3Application: myApp
            ose3TokenCredentialId:
            ose3Template: javase
            ose3TemplateParams:
              ARTIFACT_URL: ${ARTIFACT_URL}
```

For more information see [deployment.yaml](/deployment_yaml)

- *environment:* Environment to find in the deployment configuration (i.e. 'PRE')
- *submitter:* (Optional) Submitter groups. If provided group list allowed to approve ('alm-impes').
- *askTemplateSecretsParams:* (Optional) if false it dosen't ask for ose3TemplateSecretsParams credentials (true by default)

**Returns:**
- Map with the credentials selected by the user ordered by region. This map can be
used by [almOpenshiftMultiRegionDeploy](almOpenshiftMultiRegionDeploy)

```groovy

environment {
        // Deployment yaml file path
        DEPLOYMENT_YAML_FILE = 'deployment.yaml'
        // pre-pro approval submitters
        // if pre and pro submitter are different
        // use different variables
        SUBMITTERS = 'impes-group'
        // Finds given artefact in Nexus and stores the URL in the environment variable that is referenced in deployment.yaml
        ARTIFACT_URL = almMavenNexusResolver( groupId: 'com.santander.myGroup',
            artifactId: 'myArtifact',
            version: '1.0.0',
            packaging: 'jar')
        // deployment configuration
        // The yaml is read in the first
        DEPLOY_YAML = ''
        // configuration to apply for region
        // this configuration is read from approval input
        REGIONS_CONFIGURATION = ''
    }
    stages {
        stage ('Read deployment yaml') {
            agent {
                label 'ose3-deploy'
            }
            steps {
                // run your build scripts
                script {
                    // in this example the yaml contains a node base deploy
                    DEPLOY_YAML = (readYaml (file: DEPLOYMENT_YAML_FILE).deploy)
                }
            }
        }
        stage('Waiting for PRE Deployment Approval') {
                agent none
                steps {
                    // run your build scripts
                    script {
                        REGIONS_CONFIGURATION = almOpenshiftMultiRegionInput yaml: DEPLOY_YAML,
                            environment: 'PRE',
                            submitter: SUBMITTERS
                    }
                }
            }
            stage('Deploy to PRE') {
                options {
                    skipDefaultCheckout()
                }
                agent {
                    label 'ose3-deploy'
                }
                steps {
                    // deploy to all regions defined in application.yaml
                    // and overwrites the token and the artifact url
                    almOpenshiftMultiRegionDeploy configuration: DEPLOY_YAML,
                        environment: 'PRE', regionsProperties: REGIONS_CONFIGURATION
                }
            }
                // ...

```
