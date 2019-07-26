# [almMavenNexusResolver](/vars/almMavenNexusResolver.groovy)


Executes maven resolver to get uploaded artifact to nexus.

**Parameters:**
- *groupId:* Artifact group id
- *artifactId:* Artifact id
- *version:* Artifact version
- *packaging:* Packaging ('jar', 'war', ...)
- *classifier:* Classifier (optional)
- *nexusBaseUrl:* Nexus base URL (optional @see  Nexus.NEXUS_BASE_URL by default)
- *nexusMavenGroup:* Nexus maven group (optional @see  Nexus.NEXUS_MAVEN_GROUP by default)

**Returns:**
- Uploaded artifact url or throws an exeption is artifact is not found

```groovy
steps {
  println "Artifact uploaded to nexus: " + almMavenNexusResolver(
      groupId: 'com.santander.alm',
      artifactId: 'myartifact',
      version: '1.1.0-SNAPSHOT',
      packaging: 'war')
}
```

**`getArtifactReleases`**

Returns list with all release version of an artifact.

**Parameters:**
- *groupId:* Artifact group id
- *artifactId:* Artifact id
- *nexusBaseUrl:* Nexus base URL (optional Nexus.NEXUS_BASE_URL by default)
- *nexusMavenReleases:* Nexus maven group (optional Nexus.NEXUS_MAVEN_RELEASES by default)

**Returns:**
- List of versions (String) of an emtpy list if artifact was not found.

```groovy
steps {
    script {
         def versions = almMavenNexusResolver.getArtifactReleases (groupId: params.GROUP_ID,
            artifactId: params.ARTIFACT_ID)
         if (!versions) {
            error "Artifact '${params.GROUP_ID}:${params.ARTIFACT_ID}' not found in repository"
         }
         def inputSelection = input id: 'VERSION_SELECTION', 
            message: "Select a version of '${params.GROUP_ID}:${params.ARTIFACT_ID}' artifact to deploy", ok: 'OK', 
            parameters: [
                choice(choices: versions.join('\n'),
                        description: "Avaliable releases of '${params.GROUP_ID}:${params.ARTIFACT_ID}' artifact",
                name: 'VERSION')
            ]
    }
}
```  

**NOTE**: *almMavenNexusResolver.getArtifactReleases* can be executed without an agent, so It can be
used inside an input stage.