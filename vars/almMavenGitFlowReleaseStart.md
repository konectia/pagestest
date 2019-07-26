# [almMavenGitFlowReleaseStart](/vars/almMavenGitFlowReleaseStart.groovy)

Starts maven gitflow process,  merging integration and release branch. 
It also modifies the project version in the pom.xml to delete the SNAPSHOT in the release branch and increasing by one the integration SNAPSHOT version. This step, is usually done at the beginning of the build pipeline if it is a release build.

**Note**: To commit and push the changes _almMavenGitFlowReleaseEnd_ call is required.

**Parameters:**
- *integrationBranch:* Integration branch (develop by default)
- *releaseBranch:* Release branch (master by default)

```groovy

stage("SCM") {
  steps {
    // reads the pom and changes the build name with it
	script {
	  if (params.IS_RELEASE) {
        if (env.BRANCH_NAME != env.INTEGRATION_BRANCH) {
		// Error!
			error("Release process must be executed from integration branch: " + env.INTEGRATION_BRANCH )
	    }
		// merges integration branch with release branch and changes the pom version
	    almMavenGitFlowReleaseStart(env.INTEGRATION_BRANCH, env.MASTER_BRANCH)
	  }
    }
  }
}

```

**Note:** This step must be executed in *maven* type agent