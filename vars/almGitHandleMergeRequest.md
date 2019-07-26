# [almGitHandleMergeRequest](/vars/almGitHandleMergeRequest.groovy)

Performs a project git clone or a merge (if merge request is detected)

**Parameters:**
- *ffMode:* Git Fast forward mode ('NO_FF' by default). See [gitlab plugin]https://github.com/jenkinsci/gitlab-plugin) for avaliable options.

```groovy

stage ('SCM'){
  steps {
    almGitHandleMergeRequest(ffMode: 'FF')
  }
}
