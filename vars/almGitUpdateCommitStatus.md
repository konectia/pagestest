# [almGitUpdateCommitStatus](/vars/almGitUpdateCommitStatus.groovy)

Updates git commit status with current build status. Usually used in a pipeline post action.

**Parameters:**
- *name:* Build name ('build' by default).
- *commentMR:* In case of a merge request, adds a comment in the merge request with a +1 or -1 depending on the build result (boolean, true by default).

```groovy
post {
    always {
	   // updates commit status according to build result see
	  almGitUpdateCommitStatus(name: 'build')
    }
}

```
**Note: **This steps needs a connection with Gitlab (pipeline options)
```groovy
options {
   // connects the build with gitlab
   gitLabConnection(com.santander.alm.pipeline.git.Git.SERENITY_GITLAB_CONNECTION_NAME)
```