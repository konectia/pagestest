# [almGradle](/vars/almGradle.groovy)

Executes gradle task with [almGradleNexusWrapper](/almGradleNexusWrapper.md).

**Parameters:**
- *gradle:* 'gradle' or 'gradlew' command ('gradle' by default)
- *task:* Task to execute (Optional, empty by default)
- *options:* Gradle parameters to concatenate to gradle command (Optional by default empty)
- *parameters:* Parameters to add at the end of the command (Optional by default empty)

```groovy

  almGradle(gradle: 'gradlew', task: 'build')

```

**Note:** This step must be executed in *gradle* type agent