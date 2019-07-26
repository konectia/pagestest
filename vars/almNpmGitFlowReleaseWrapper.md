# [almNpmGitFlowReleaseWrapper](/vars/almNpmGitFlowReleaseWrapper.groovy)

Executes a NPM git flow release process if  is release var is set. If release, first
versioning process is executed, then build process indicated in body, and finally commit and push.

**Parameters:**
- *body:* Body to execute
- *release:* if release process should be executed (false by default)
- *integrationBranch:* Integration branch (develop by default)
- *releaseBranch:* Release branch (master by default)

```groovy
almNpmGitFlowReleaseWrapper({
  npm build
  }, env.BRANCH_NAME == 'master')
```

**Note:** This step must be executed in *nodejs* or *nodejs-v8* type agent