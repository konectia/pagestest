# [almInputCredentialsList](/vars/almInputCredentialsList.groovy)

 Includes an user input in the pipeline that shows a list with all current user credentials.
 Usually, this step is used before a pre or pro promotion approval to select a deployment credential.

**Parameters:**
- *id:* Input name (required)
- *message:* input message (required)
- *submitter:* list of approvers separated by ',' (null by default)
- *time:* Time out (int). If defined a time-out is added (null by default)
- *unit:* Time out unit (required if time paremeter is provided. Null by default). Avaliable options are: 'SECONDS','MINUTES', 'HOURS', 'DAYS'...
- *privateCredentials:* (Deprecated) Not used
- *submitterParameter:* (String) If specified, this is the name of the return value that will contain the ID of the user that approves this input. The return value will be handled in a fashion similar to the parameters value.
- *credentialType:* (String) If specified, only this credential types are listed. For more information see 
(https://github.com/jenkinsci/credentials-plugin). Example of types are: 
  * _com.cloudbees.jenkins.plugins.sshcredentials.impl.BasicSSHUserPrivateKey_: [SSH Credential](https://github.com/jenkinsci/ssh-credentials-plugin)
  * _com.cloudbees.plugins.credentials.impl.UsernamePasswordCredentialsImpl_: User password credentials (https://github.com/jenkinsci/credentials-plugin)
   
```groovy
stage ('Promote') {
  agent none
  steps {
    script {
      credId = almInputCredentialsList(
          id: 'promote',
          message: 'Select a deployment credential',
          submitter: 'idUserOrGroup',
          credentialType: 'com.cloudbees.jenkins.plugins.sshcredentials.impl.BasicSSHUserPrivateKey')
      echo "Using credentials: $credId"
    }
  }
}
```

**NOTE:** Agent none should be set in input stages so no agent resources are locked until input is submitted.