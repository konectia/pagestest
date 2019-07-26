# [almOpenshiftDeploy](/vars/almOpenshiftDeploy.groovy)

Executes a deploy of an OpenShift application into given project.

**Parameters:**
- *ose3Region:* PaaS region URI (_e.g_: `https://api.boaw.paas.gsnetcloud.corp:8443`)
- *ose3Project:* PaaS project name (_e.g_: `project-dev`)
- *ose3Application:* Application name (_e.g_: `username_serv`)
- *ose3Template:* OSE3 Template name (_e.g_: `javase`)
- *ose3TokenCredentialId:* OpenShift deploy token in project Jenkins credential identifier.
- *ose3TemplateParams:* Map of the parameters for the OSE3 Template (Optional)
- *ose3TemplateSecretsParams:* Map of the secret parameters for the OSE3 Template (Optional). These are template parameters that are secrets (i.e. DB_PASS=myPass). These parameters search the credential id in jenkins (secret text type) and its value is printed as (DB_PASS=****)
- *ose3BlueGreen:* Blue green deployment flag (boolean) (Optional, `false` by default). Set to `true` if the deployment is in the shadow environment.
- *ose3Rollback:* Rollback deployment on failure flag (boolean) (Optional, `true` by default). Set to `false` if no rollback should be carried out on deployment failure.


```groovy
stage ('Deploy') {
  agent {
    label 'ose3-deploy'
  }
  steps {
    almOpenshiftDeploy(
        ose3Region: 'https://api.boaw.paas.gsnetcloud.corp:8443',
        ose3Project: 'project-dev',
        ose3Application: 'myapp',
        ose3Template: 'javase',
        ose3TokenCredentialId: 'Deploy-In-Dev-Credential-Id',
        ose3TemplateParams: [ARTIFACT_URL: 'http://nexus/myArtifact']
    )
  }
}
```

**NOTE:** This step must be executed in an *ose3-deploy* agent.
