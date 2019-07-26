# [almMavenGitFlowReleaseEnd](/vars/almMavenGitFlowReleaseEnd.groovy)

Ends maven gitflow process pushing changes from integration branch to release branch. This step, is usually done at the end of the build pipeline if it is a release build.

**Note:** This step must be executed after _[almMavenGitFlowReleaseStart](/almMavenGitFlowReleaseStart.md)_.

**Parameters:**
- *integrationBranch:* Integration branch (develop by default)
- *releaseBranch:* Release branch (master by default)

```groovy
stage('GitFlow') {
  when {
    // When release is launched commits changes
	expression { return params.IS_RELEASE }
  }
  steps {
    almMavenGitFlowReleaseEnd(env.INTEGRATION_BRANCH, env.MASTER_BRANCH)
  }
}
```

**Note:** This step must be executed in *maven* type agent