# apic

Provides a collection of methods for working with API Connect APIs

## Steps for Continuous Integration

The support for crafting continuous integration pipelines for *API Manager* **APIs** has been provided via the following steps:

**`declareApi`**

Required for defining the API that will be handled by the other continuous integration and delivery tasks.

The step checks that the specified file exists, it can be parsed as a YAML file, and is defined as a Swagger YAML file, with defined `info.title` and `info.version` properties.

  - Parameters:
    - _file:_ (Required) API definition file
    - _description:_ (Required) API description
  - Returns:
      - ApiDefinition

```groovy
  steps {
    script {
      def api = apic.declareApi file: 'swagger.yaml', description: "My API"
    }
  }
```

**`declareProduct`**

Required for defining the PRODUCT that will be handled by the other continuous integration and delivery tasks.

The step checks that the specified file exists, it can be parsed as a YAML file, and is defined as a Swagger YAML file, with defined `info.title` and `info.version` properties.

  - Parameters:
    - _file:_ (Required) API definition file
  - Returns:
      - ProductDefinition

```groovy
  steps {
    script {
      def api = apic.declareProduct file: 'swagger.yaml'
    }
  }
```

**`validateApi`**

The step carries out the validation of a declared API, leveraging the `apic` client `validate` functionality. This step must be executed within an `apic` agent.

- Parameters:
  - api:_ (Required) API definition

```groovy
  steps {
    script {
      apic.validateApi api: api
    }
  }
```

**`getApisFromProduct`**

Get the apis referenced in the yaml of a PRODUCT.

  - Parameters:
    - product:_ (Required) PRODUCT definition
  - Returns:
    - List [api_name:api_version]

```groovy
  steps {
    script {
      def list_apis = apic.getApisFromProduct product: product
    }
  }
```

**`downloadApisFromProduct`**

Download from Nexus a list of apis.

  - Parameters:
    - groupId:_ (Required) Nexus GroupId from where to download the apis
    - apis:_ (Required) Api list
    - product:_ (Required) PRODUCT definition

```groovy
  steps {
    script {
      apic.downloadApisFromProduct groupId: 'com.santander', apis: list_apis, product: product
    }
  }
```


**`validateProduct`**

The step carries out the validation of a declared PRODUCT, leveraging the `apic` client `validate` functionality. This step must be executed within an `apic` agent. It is necessary download the yaml of the apis.

- Parameters:
  - product:_ (Required) PRODUCT definition

```groovy
  steps {
    script {
      apic.validateProduct product: product
    }
  }
```

**`sonarQube`**

The step analyzes the project with *SonarQube*. The user may decide whether or not the results are to be published within **SonarQube** or in **Jenkins**.

- Parameters:
  - _api:_ (Required) API definition
  - _instance:_ (Required) target SonarQube instance. For the time being only the `ISBAN` instance provides API-related analysis support
  - _groupId:_ (Required) `groupId` for identifying the API definition
  - _artifactId:_ (Required) `artifactId` for identifying the API definition
  - _mode:_ (Required) analysis mode. It can be one of `publish` or `preview` depending on whether the result is to be published in SonarQube or kept as local analysis that will be attached to the build results.

```groovy
  steps {
    script {
      apic.sonarQube api: api, instance: 'ISBAN', groupId: 'com.santander', artifactId: 'example-api', mode: 'publish'
    }
  }
```

## Steps for Continuous Delivery

To help with continuous delivery tasks, the following steps are available:

**`uploadToNexus`**

The step publishes in *Nexus* an API bundle for a declared API leveraging the **ALM Global API Maven Archetype**. The step requires specifying the `groupId`, `artifactId`, and `version` (*a.k.a.*, GAV) to be used for publishing the bundle. This step must be executed within a `maven` agent.

- Parameters:
  - _api:_ (Required) API definition
  - _groupId:_ (Required) `groupId` for identifying the API definition
  - _artifactId:_ (Required) `artifactId` for identifying the API definition
  - _version:_ (Required) `artifactId` for identifying the API definition. It should end with `-SNAPSHOT` if the version is not a release.

```groovy
  steps {
    script {
      apic.uploadToNexus api: api, groupId: 'com.santander', artifactId: 'example-api', version: '1.0.0'
    }
  }
```


**`releaseInput`**

The step enables asking the user which are the release version and next development version to be generated, using as input the current version of a declared API. This step should be carried on release builds only.

- Parameters:
  - _api:_ (Required) API definition. The step expects the API version to follow the SemanticVersion format.
- Returns
  - Map (*e.g.*, `[ release: "1.0.0", development: "1.0.1" ]`)

```groovy
  steps {
    script {
      def versions = apic.releaseInput api: api
    }
  }
```

**`startGitFlow`**

The step takes care of carrying out the local actions for generating a new release of a declared API according to the **Git Flow** branching model.

- Parameters:
  - _masterBranch:_ (Required) master branch
  - _developmentBranch:_ (Required) integration branch
  - _api:_ (Required) API definition.
  - _releaseVersion:_ (Required) the version to be released. It must follow the SemanticVersion format. It must also be lower than the `developmentVersion`.
  - _developmentVersion:_ (Required) the next development version. It must follow the SemanticVersion format. It must also be greater than the `releaseVersion`.

```groovy
  steps {
    script {
      apic.startGitFlow masterBranch: 'master', integrationBranch: 'develop', api: api, releaseVersion: '1.0.0', developmentVersion: '1.0.1'
    }
  }
```

**`endGitFlow`**

The step takes care pushing to the remote repository the actions carried out by the previous step.

- Parameters:
  - _masterBranch:_ (Required) master branch
  - _developmentBranch:_ (Required) integration branch

```groovy
  steps {
    script {
      apic.endGitFlow masterBranch: 'master', integrationBranch: 'develop'
    }
  }
```

## Steps for Continuous Deployment

Finally, to deal with continuous deployment duties, the following steps have been added:

*`deploymentConfigurationInput`*

The step enables asking the user the password for logging into the API connect instance where an API will be deployed. A list of approvers may be specified to restrict who can approve a deployment. In addition, if variable resolution is required, the version of the property file can be requested as well.

- Parameters:
  - _approvers:_ (Required) list of approvers. If empty, any user can approve the deployment (provide the credentials for carrying out the process).
  - _version:_ (Optional) proposed variable resolution configuration file version.
- Returns
  - Map (*e.g.*, `[ version: '1.0.1', credentials: [ Username: 'xi123456', Password: 's3cr3t' ] ]`)

```groovy
  steps {
    script {
      def configuration=apic.deploymentConfigurationInput version: '1.0.1', approvers: [ 'xi123456', 'n12345' ]
    }
  }
```

**`downloadBundle`**

The step enables downloading an API bundle for a given API, provided the GAV used for publishing it (see `uploadToNexus` step).

- Parameters:
  - _groupId:_ (Required) `groupId` for identifying the API bundle
  - _artifactId:_ (Required) `artifactId` for identifying the API bundle
  - _version:_ (Required) `artifactId` for identifying the API bundle
- Returns:
  - ApiBundle

```groovy
  steps {
    script {
      def bundle=apic.downloadBundle groupId: 'com.santander', artifactId: 'example-api', version: '1.0.0'
    }
  }
```

**`gitlabConfiguration`**

The step enables providing a *variable map* to be used in the variable resolution process backed by a property file from a **GitLab** instance.

- Parameters:
  - _gitlabHost:_ (Required) Base URL of the GitLab configuration where the configuration file is stored.
  - _configPath:_ (Required) Path to the particular configuration file.
  - _credentials:_ (Required) Credentials for the retrieval of the configuration file. It should be a Map with `Username` and `Password` properties.
- Returns:
  - ConfigurationProvider

```groovy
  steps {
    script {
      def provider=apic.gitlabConfiguration gitlabHost: 'https://serenity-gitlab.almdes01.gsnetcloud.corp', configPath: "/serenity-alm/apic-sample-catalogation-project/raw/"+configuration.version+"/config/environment.dev.properties", credentials: configuration.credentials
  }
}
```

**`fileConfiguration`**

The step enables providing a *variable map* to be used in the variable resolution process backed by a property file from the local repository.

- Parameters:
  - _file:_ (Required) Local path to a file within the workspace to the configuration file. The file must exist.
- Returns:
  - ConfigurationProvider

```groovy
  steps {
    script {
      def provider=apic.fileConfiguration file: 'config/environment.' + ((env.BRANCH_NAME == 'develop') ? 'dev' : 'cert') + '.properties'
  }
}
```

**`resolveVariables`**

The step substitutes a set of `$#`-delimited variables defined in the API definition included in a given API bundle by according to a provided *variable map* (see steps above).

- Parameters:
  - _bundle:_ (Required) The API bundle that includes the API definition to be resolved.
  - _provider:_ (Required) The ConfigurationProvider that will provide the *variable map* for resolving the variables of the target API definition.

```groovy
  steps {
    script {
      apic.resolveVariables bundle: bundle, provider: provider
  }
}
```

**`publishProduct`**

The step publishes the PRODUCT definition included in a given API bundle in a **API Manager** instance. The PRODUCT definition will be published in the catalog, organization and space indicated. This action is carried out leveraging the `apic` client.

- Parameters:
  - product:_ (Required) PRODUCT definition
  - apiServerUrl:_ (Required) Host name/ip and port of the *API Manager* instance where the PRODUCT definition will be published.
  - apiOrganization:_ (Required) Organization within the specified *API Manager* instance where the PRODUCT definition will be published.
  - credentials:_ Credentials for login into the specified *API Manager*. It can be a Map with `Username` and `Password` properties, or a string with the identified of a global username credential.
  - apiCatalog:_ (Required) Catalog within the specified *API Manager* instance where the PRODUCT definition will be published.
  - apiSpace:_ (Required) Organization within the specified *API Manager* instance where the PRODUCT definition will be published.

```groovy
  steps {
    script {
      apic.publishProduct product: product, apiServerUrl: 'apim-dev-global.scisb.isban.corp:443', 
              apiOrganization: 'soporte-al-ciclo', credentials: 'userpassCredential',
              apiCatalog: 'dev', apiSpace: 'intraclie'
  }
}
```

**CAVEATS**

Steps 2 to 6 **MUST** be run in the same `apic` agent, as the devised deployment workflow relies on local storage.

**Note:** This step must be executed in *apic* type agent