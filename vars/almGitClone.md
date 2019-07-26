# [almGitClone](/vars/almGitClone.groovy)

Performs a project git clone (checkout scm)

**Parameters:** (None)

```groovy

stage ('SCM'){
  steps {
    almGitClone()
  }
}

```