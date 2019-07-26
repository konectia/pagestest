#sshInputAgent

 Wraps 'sshagent' pipeline step to allow to use user-scoped credentials of the last input approver
 The step reference is the same as 'sshagent' but renaming to 'sshInputAgent'
 See [sshagent step reference](https://jenkins.io/doc/pipeline/steps/ssh-agent/#code-sshagent-code-ssh-agent)

```groovy
stage ('Promote') {
  steps {
    sshInputAgent (credentials: ['deploy-dev-id']) {
      sh 'ssh -o StrictHostKeyChecking=no -l cloudbees 192.168.1.106 uname -a'
    }
  }
}