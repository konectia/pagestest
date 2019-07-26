# [ansible](/vars/ansible.groovy)

Executes an ansible playbook.

**Parameters:**

- *ansibleCredentialId:* Credential to connect to linux machine by ssh (SSH or user password)
- *winrmCredentialId:* User-password credential to connect to windows machine by winrm
- *vaultCredentialId:* Credential id for vaults. Must be of 'Secret file' type.
- *playbook:* Name playbook to execute ('site.yml' by default)
- *inventory:* Inventory of machines to provision (host by default)
- *ansibleGalaxy:* If true, it will install required ansible-galaxy roles defined in requiremets
- *requirements:* File define ansible roles ('requirements.yml' by default)
- *git:* If defined git repository with ansible project otherwise the ansible project must be in the workspace
- *gitBranch:* If git project is defined, Git branch (git default branch is used by default)
- *gitCredentialId:* If git project is defined, Git credential id (by default uses the same of the project)
- *gitCloneDirectory:* If defined, directory to clone the git project otherwise the ansible project is cloned in a subfolder
of the workspace
- *extraParams:* Extra variables

```groovy
stage ('Deploy') {
  agent {
    label 'ansible'
  }
  steps {
    ansible(
      ansibleCredentialId: 'ssh-credential-id',
      ansibleVaultCredentialId: 'vault_file',
      git: 'ssh://git@gitlab.alm.gsnetcloud.corp:2220/serenity-alm/ansible-roles/op-war-ansible.git',
      inventory: 'inventories/cert/hosts',
      ansibleGalaxy: true,
      extraParams: [
        webapps: [['url': '${ARTIFACT_NEXUS_URL}', 'path': '/example']]
      ]
    )
  }
}
```

**NOTE:** This step must be executed in an *ansible* agent.
