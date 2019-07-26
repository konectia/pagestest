# [almOpenshiftSwitch](/vars/almOpenshiftSwitch.groovy)

Executes a switch inOpenshift of an application deployed with blue-green flag set to true

**Parameters:**
- *ose3Region:* PaaS region URI (e.g: host:port)
- *ose3Project:* PaaS project name (e.g: (project-dev)
- *ose3Application:* Application name (e.g: username_serv)
- *ose3TokenCredentialId:* OpenShift deploy token in project Jenkins credential id.

```groovy
stage ('Deploy') {
  agent {
    label 'ose3-deploy'
  }
  steps {
    almOpenshiftSwitch(
        ose3Region: 'https://api.boaw.paas.gsnetcloud.corp:8443',
        ose3Project: 'project-dev',
        ose3Application: 'myapp',
        ose3TokenCredentialId: 'Deploy-In-Dev-Credential-Id'
    )
  }
}
```

**Note:** This step must be executed in *ose3-deploy* type agent