# [almOpenshiftMultiRegionSwitch](/vars/almOpenshiftMultiRegionSwitch.groovy)

Executes a switch of an OpenShift application deployed with the blue-green deploy given project.

* The parameter the must be passed insider the are:

**Parameters:**
- *configuration:* Deployment configuration (see [deployment.yaml](/deployment_yaml))
- *regionsProperties:* Environment map with configuration to apply to the configuration see alm [almOpenshiftMultiRegionInput](almOpenshiftMultiRegionInput.md)
- *environment:* Name of the environment to deploy
- *parallel:* parallel (optional if true al regions are deployed in parallel, true by default)

**NOTE**: Configuration and environment or environmentMap arguments are required


```groovy

stage('Switch to PRO') {
    agent {
        label 'ose3-deploy'
    }
    steps {
       almOpenshiftMultiRegionSwitch  configuration: DEPLOY_YAML,
            environment: 'PRO', regionsProperties: REGIONS_CONFIGURATION
    }     
}        

```

**INFO:** This step supports environment variables in the yaml.

**Note:** This step must be executed in *ose3-deploy* type agent