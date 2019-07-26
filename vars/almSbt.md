# [almSbt](/vars/almSbt.groovy)

Executes given Scala Build Tool goal

**Parameters:**
- *param:* goal sbt goal ('clean test' for example)
- *options:* If additional sbt parameter is required (empty by default)
- *repos:* Array of repositories to be added to ~/.sbt/repositories config file. Must follow this format: [[name: "repo1", context: "contex1", opts: ""],[name: "repo2", context: "contex2", opts: "options without ,"],...]. Context is mean to be last context param in nexus repo url

```groovy
stage ('SBT Build') {
  agent {
    label 'sbt'
  }
  steps {
    almSbt(goal: 'compile')
  }
}
```

Another example:

```groovy
def extraRepos = [
  [name: "santander-maven-plugin",  context: "maven-public"],
  [name: "santander-maven-central", context: "maven-central"],
  [name: "BintraySBTPluginReleases",context: "BintraySBTPluginReleases", opts: ""]
]

pipeline{
    agent{
        label 'sbt'
        }
    stages{
        stage('Build'){
            steps{
                almSbt(goal: "clean test:compile", repos: extraRepos)
            }
        }
    }
}
```

**Note:** This step must be executed in *sbt* type agent