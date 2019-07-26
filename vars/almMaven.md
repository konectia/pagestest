# [almMaven](/vars/almMaven.groovy)

Executes a maven goal.

**Parameters:**
- *goal:* Maven goal to execute ('clean verify' for example)
- *options:* If additional maven parameter is required (empty by default)

```groovy
stage("Tests") {
  steps {
    almMaven(goal: 'integration-test')
  }
  post {
    always {
      junit '**/target/*-reports/TEST-*.xml'
    }
  }
}
```

**Note:** This step must be executed in *maven* type agent