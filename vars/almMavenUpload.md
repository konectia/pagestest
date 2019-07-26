# [almMavenUpload](/vars/almMavenUpload.groovy)

Executes maven upload (mvn deploy) to nexus snapshots repository if it version ends with SNAPSHOT or
 otherwise to release repository

**Parameters:**
- *pom:* Route to pom in workspace ('pom.xml' by default)
- *arguments:* maven arguments (optional '-Dmaven.test.skip=true' by default)
- *packagingExtension:* Required when the packaging is not the same with the artifact extension
**Returns:**
- root's artifact uploaded artifact url (in multimodule projects, to get each module artifact
URL you can call almMavenNexusResolver step.

```groovy
stage ('Upload') {
  steps {
    script {
      def artifactUrl = almMavenUpload()
      println "Artifact URL = $artifactUrl"
    }
  }
}
```

**Note:** If you need to save the result into variable you need an script section, otherwise you can
call the step without script section.

**Note:** This step must be executed in *maven* type agent