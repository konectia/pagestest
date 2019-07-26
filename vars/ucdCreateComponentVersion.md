# [ucdCreateComponentVersion](/vars/ucdCreateComponentVersion.groovy)

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
    ucdCreateComponentVersion( server: 'https://ucd-server.santander.corp:8443', 
        username: 'user', password: 'secret', 
        component:  "${GROUP_ID}:${ARTIFACT_ID}", 
        componentVersion: "${VERSION}-${BUILD_ID}", 
        versionProperties: [ARTIFACT_URL: "${ARTIFACT_NEXUS_URL}"] )ucdCreateComponentVersion.md
}
```

This step is equivalent to [ucd.createComponentVersion](/vars/ucd) step