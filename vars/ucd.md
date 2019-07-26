# [ucd](/vars/ucd.groovy)

Provides a collection of methods for working with Urban Code Deploy

## createComponentVersion

 Creates a new component version in UCD. The component must exist in UCD.
   - Parameters:
      - _server:_ (Required) UCD server URL (i.e. 'https://ucd-server.santander.corp:8443')
      - _username:_ (Required) UCD connection user name
      - _password:_ (Required) UCD connection password
      - _component:_ (Required) UCD Component name  (i.e. 'com.santander.alm.mycomponent')
      - _componentVersion:_ (Required) UCD Component version name to create (i.e. '1.0.0')
      - _versionProperties:_ (Optional) Map of version properties (i.e. [ARTIFACT_URL: 'https://nexus.alm.com/maven-releases/myartifact.zip'])

```groovy
steps {
    script {
       ucd.createComponentVersion( server: 'https://ucd-server.santander.corp:8443', 
           username: 'user', password: 'secret', 
           component:  "${GROUP_ID}:${ARTIFACT_ID}", 
           componentVersion: "${VERSION}-${BUILD_ID}", 
           versionProperties: [ARTIFACT_URL: "${ARTIFACT_NEXUS_URL}"] )
    }
}
```

## getComponentVersions

Gets all component versions in UCD. The component must exist in UCD.

   - Parameters:
      - _server:_ (Required) UCD server URL (i.e. 'https://ucd-server.santander.corp:8443')
      - _username:_ (Required) UCD connection user name
      - _password:_ (Required) UCD connection password
      - _component:_ (Required) Component name  (i.e. 'com.santander.alm.mycomponent')
      - _inactive:_ (Optional boolean) if true also inactive components are retrieved (default false)
   - Returns:_
      - List with component versions (List<String> i.e. ['1.0.1-SNAPSHOT-1', '1.0.0'])

```groovy
steps {
    script {
       def versions = ucd.getComponentVersions( server: 'https://ucd-server.santander.corp:8443', 
           username: 'user', password: 'secret', 
           component:  "${GROUP_ID}:${ARTIFACT_ID}")
       echo ("Component versions: ${versions}")
    }
}
```
      
## deployApplication

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
    script {
        ucd.deployApplication (server: 'https://ucd-server.santander.corp:8443', 
            username: 'user', password: 'secret', 
            application: 'my-ucd-application', environment: 'CERT', 
            process: 'my-ucd-application-process', 
            componentVersions: [( "${GROUP_ID}:${ARTIFACT_ID}"): ["${VERSION}-${BUILD_ID}"]])
    }
}
```