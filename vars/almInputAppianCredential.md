# [almInputAppianCredential](/vars/almInputAppianCredential.groovy)

Approval input that waits for deployment approval support.
This step, returns the given information provided by the approvers.

**Parameters:**
- *approvers:* list of approvers separated by ',' (null by default)

**Returns:**
- Map with the credentials selected by the user ordered: Username, Password and GitCredentialId.

```groovy

environment {
	// pre-pro approval approvers
	// if pre and pro approver are different
	// use different variables
	APPROVERS = ['impes-developer','impes-technical-lead','impes-product-owner','alm-developer']
}
stage('Promotion check') {
	agent none
	steps {
		script {
			userInput = almInputAppianCredential(APPROVERS)
		}
	}
}

```

**NOTE:** Agent none should be set in input stages so no agent resources are locked until input is submitted.
