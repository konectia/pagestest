#[almSonar]

Executes sonar analysis.

**Parameters:**
- *deploymentID:* Identify the Jenkins credential ID for the binary repository (Optional)
- *sonarInstanceName:* (String)if given this sonar instance is used (DEFAULT_SONAR_INSTANCE_NAME by default)
- *analisysMode:* (String) sonar scan mode. If given -Dsonar.analysis.mode argument is added. Valid values are: 'publish', 'preview' or 'issues'
- *publishHtml:* (Boolean) if true sonar result is archived (true by default)
- *sonarProperties:* (Map) if given all properties are added to command (default empty)


```groovy
stage("SonarQube") {
  steps {
    almSonar(analisysMode: (env.BRANCH_NAME == 'development') ? '' : 'preview')
  }
}
```