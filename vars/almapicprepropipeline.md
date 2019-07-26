---
title: This is my title
layout: home
---
# [almApicPreProPipeline](/vars/almApicPreProPipeline.groovy)

Executes a  promotion pipeline for API Manager APIs with approval and deployment in PRE and PRO environments. 

This pipeline creates following steps:

![Api PRE-PRO Pipeline](./../resources/images/apiPreProPipeline.jpg)

**Stages:**
 * **Deployment approve to PRE:**  In this stage to be requested the user the configuration to be used for resolving the variables defined within the API definition as well as the password for connecting to the _API Manager_ were the resolved API definition will be deployed in the PRE environment.
 * **Deploy PRE:** In this stage, to be downloaded the API bundle to be deployed, retrieve the configuration from GitLab, resolve the variables defined in the bundled API definition with the retrieved configuration, and finally deploy the resolved API definition to the target _API Manager_ instance in PRE.
 * **Deployment approve to PRO:**  In this stage to be requested the user the configuration to be used for resolving the variables defined within the API definition as well as the password for connecting to the _API Manager_ were the resolved API definition will be deployed in the PRO environment.
 * **Deploy PRO:**  In this stage, to be downloaded the API bundle to be deployed, retrieve the configuration from GitLab, resolve the variables defined in the bundled API definition with the retrieved configuration, and finally deploy the resolved API definition to the target _API Manager_ instance in PRO.
 * **POST:**  Post deploy stage to define deploy actions. 

**Parameters:**
* **Global Pipeline parameters**:
    * *approversPre:* List of approvers authorized to deploy in the PRE environment.  If empty, any user can approve the deployment (provide the credentials for carrying out the process).
    * *approversPro:* List of approvers authorized to deploy in the PRO environment.  If empty, any user can approve the deployment (provide the credentials for carrying out the process).
    * *GROUP_ID*: groupId for identifying the API bundle
    * *ARTIFACT_ID*: artifactId for identifying the API bundle
    * *ARTIFACT_VERSION*: artifact version for identifying the API bundle

* **Deploy** ('deploy'):
    * *pre*: Variables required to deploy in the PRE environment
       * *GITLAB_HOST*: Base URL of the GitLab configuration where the configuration file is stored.
       * *CONFIG_PATH*: Path to the particular configuration file to PRE environment.
       * *API_SERVER_URL*: Host name/ip and port of the _API Manager_ instance where the API definition will be published.
       * *API_ORGANIZATION*: Organization within the specified _API Manager_ instance where the API definition will be published.
    * *pro*: Test results to archive (_''target/**/TEST-*.xml'_ by default)
       * *GITLAB_HOST*: Base URL of the GitLab configuration where the configuration file is stored.
       * *CONFIG_PATH*: Path to the particular configuration file to PRO environment.
       * *API_SERVER_URL*: Host name/ip and port of the _API Manager_ instance where the API definition will be published.
       * *API_ORGANIZATION*: Organization within the specified _API Manager_ instance where the API definition will be published.

## Examples:

```groovy

library 'global-alm-pipeline-library'
	almApicPreProPipeline (
		approversPre: ['impes-developer','impes-technical-lead','impes-product-owner','alm-developer'],
		approversPro: ['impes-developer','impes-technical-lead','impes-product-owner','alm-developer'],
		GROUP_ID: 'com.santander.alm',
		ARTIFACT_ID: 'customer-relationship-api',
		ARTIFACT_VERSION: '2.0.0', 
		deploy: [
			pre: [
				GITLAB_HOST: 'https://serenity-gitlab.almdes01.gsnetcloud.corp',
				CONFIG_PATH: '/xIS07723/apic-sample-catalogation-project/raw/master/config/environment.pre.properties',
				API_SERVER_URL: 'apim-pre-global.scisb.isban.corp:443',
				API_ORGANIZATION: 'soporte-al-ciclo'				
			],
			pro: [
				GITLAB_HOST: 'https://serenity-gitlab.almdes01.gsnetcloud.corp',
				CONFIG_PATH: '/xIS07723/apic-sample-catalogation-project/raw/master/config/environment.pro.properties',
				API_SERVER_URL: 'apim-pro-global.scisb.isban.corp:443',
				API_ORGANIZATION: 'soporte-al-ciclo'
			]
		],
		post: [
			agent: 'apic',
			body: { pipeline ->
				echo "Ejecuntado POST FINAL"
			}
		]
	)
    
```