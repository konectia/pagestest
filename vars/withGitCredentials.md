# [withGitCredentials](/vars/almGitClone.groovy)

Executes the body (usually git commands) setting configured project credentials. 
Supports SSH and HTTP credentials.

**Parameters:** (None)

```groovy

stage ('Tag'){
  steps {
    withGitCredentials {
        sh 'git tag -a v1.4 -m "my version 1.4"'
        sh 'git push --tags'
    }
  }
}

```