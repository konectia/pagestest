#withLastApproverCredentials  

 Wraps 'withCredentials' pipeline step to allow to use user-scoped credentials of the last input approver
 The step reference is the same as 'withCredentials' but renaming to 'withLastApproverCredentials'
 See [withCredentials step reference](https://jenkins.io/doc/pipeline/steps/credentials-binding/#code-withcredentials-code-bind-credentials-to-variables)

```groovy
stage ('Promote') {
  steps {
    withCredentials([usernameColonPassword(credentialsId: 'mylogin', variable: 'USERPASS')]) {
      sh '''
        set +x
        curl -u "$USERPASS" https://private.server/ > output
      '''
    }
  }
}
```
