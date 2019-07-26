# [almGradleParseProperties](/vars/almGradleParseProperties.groovy)

Executes the 'properties' gradle task, and returns a map with all properties defined in the gradle file.

Example:

```groovy
steps {
  script {
    gradleProps = almGradleParseProperties()
    currentBuild.displayName = gradleProps['name'] + ":" + gradleProps['version']
  }
}

```

**Note:** This step must be executed in *gradle* type agent