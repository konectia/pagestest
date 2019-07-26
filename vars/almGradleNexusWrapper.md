# [almGradleNexusWrapper](/vars/almGradleNexusWrapper.groovy)

Wraps gradle execution to provide nexus credentials. This step, takes as argument a closure to be wrapped.

```groovy

  almGradleNexusWrapper {
    sh 'gradle build'
  }

```

**Note:** This step must be executed in *gradle* type agent