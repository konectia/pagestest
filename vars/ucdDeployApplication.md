# [ucdDeployApplication](/vars/ucdDeployApplication.groovy)

Deploys an application in UCD. The application, process, environment an component must exist in UCD.

- Parameters:
   - Parameters:
      - _server:_ (Required) UCD server URL (i.e. 'https://ucd-server.santander.corp:8443')
      - _username:_ (Required) UCD connection user name
      - _password:_ (Required) UCD connection password
      - _application:_ (Required) UCD Application name
      - _process:_ (Required) UCD application process
      - _environment:_ (Required) UCD Deployment environment
      - _componentVersions:_ (Required) Map of UCD component versions to deploy (i.e. '[comp1: ['1.0']])
      - _description:_ (optional) Description of this request. Default ''
      - _snapshot:_ (optional boolean) if true an snaphot is taken. default ''
      - _onlyChanged:_ (optional boolean) if true only changed files are deployed (default false)
      - _waitForApplicationProcess:_ (optional boolean) if true waits until deployment finishes (default true)

```groovy
steps {    
    ucdDeployApplication (server: 'https://ucd-server.santander.corp:8443', 
        username: 'user', password: 'secret', 
        application: 'my-ucd-application', environment: 'CERT', 
        process: 'my-ucd-application-process', 
        componentVersions: [( "${GROUP_ID}:${ARTIFACT_ID}"): ["${VERSION}-${BUILD_ID}"]])
}
```

This step is equivalent to [ucd.deployApplication](/vars/ucd) step