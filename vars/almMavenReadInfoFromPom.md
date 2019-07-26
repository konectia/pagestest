# [almMavenReadInfoFromPom](/vars/almMavenReadInfoFromPom.groovy)

Returns a map with pom info.

**Parameters:**
- *pom:* Route to pom in workspace ('pom.xml' by default)
**Returns:**
- Map with pom info

| Property                | Description                                             |
|:------------------------|:--------------------------------------------------------|
| POM_ARTIFACTID          | Artifact's id                                           |
| POM_GROUPID             | Artifact's group id                                     |
| POM_VERSION             | Artifact's version                                      |
| POM_PACKAGING           | Artifact's packaging type or 'jar' if it is not defined |
| POM_PROJECT_NAME        | Project's name                                          |
| POM_PROJECT_DESCRIPTION | Project's description                                   |

```groovy

stage ('SCM') {
  steps {
    almGitClone()
    script {
      def pomMap = almMavenReadInfoFromPom()
      currentBuild.displayName = "${pomMap['POM_GROUPID']}:${pomMap['POM_ARTIFACTID']}:${pomMap['POM_VERSION']}-${env.BUILD_ID}"
    }
  }
}

```

**Note:** This step must be executed in *maven* type agent