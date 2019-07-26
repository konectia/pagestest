# [ucdCreateComponentVersion](/vars/ucdCreateComponentVersion.groovy)

Retrieves all component versions in UCD. The component must exist in UCD.

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

This step is equivalent to [ucd.createComponentVersion](/vars/ucd) step