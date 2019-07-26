# [almOpenshiftMultiRegionDeploy](/vars/almOpenshiftMultiRegionDeploy.groovy)

Executes a deploy of an OpenShift application into given project.

* The parameter the must be passed insider the are:

**Parameters:**
- *configuration:* Deployment configuration (see [deployment.yaml](/deployment_yaml))
- *regionsProperties:* Environment map with configuration to apply to the configuration see alm [almOpenshiftMultiRegionInput](/vars/almOpenshiftMultiRegionInput)
- *environment:* Name of the environment to deploy
- *parallel:* parallel (optional if true al regions are deployed in parallel, true by default)

**NOTE**: Configuration and environment or environmentMap arguments are required


```groovy

stage('Deploy to PRE') {
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

```

**INFO:** This step supports environment variables (defined in environment pipeline section) in the yaml.

See /resources/examples/declarative

**Note:** This step must be executed in *ose3-deploy* type agent
