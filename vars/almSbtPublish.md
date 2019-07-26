# [almSbtPublish](/vars/almSbtPublish.groovy)

Publishes the artifact referenced at the sbt build file to repository.
If artifact version is a snapshot, artifact is uploaded to snapshot Maven repository
 else to release Maven repository.
The deployment repository is defined by the slave configuration

**Parameters:**
- *deploymentID:* Identify the Jenkins credential ID for the binary repository (Optional)

```groovy

stage ('SBT Publish') {
  agent {
    label 'sbt'
  }
  steps {
    almSbtPublish()
  }
}

```

**Note:** This step must be executed in *sbt* type agent