### [almInput](/vars/almInput.groovy)

 Includes an user input in the pipeline.

**Parameters:**
- *id:* Input name (required)
- *message:* input message (required)
- *parameters:* Input parameters (null by default)
- *submitter:* list of approvers separated by ',' (null by default)
- *time:* Time out. If defined a time-out is added (null by default)
- *unit:* Time out unit (required if time paremeter is provided. Null by default). Avaliable options are: 'SECONDS','MINUTES', 'HOURS', 'DAYS'...

```groovy
stage ('Promote') {
  agent none
  steps {
    almInput(id: 'promote', message: 'Promote?', submitter: 'impes')
  }
}
```

**NOTE:** Agent none should be set in input stages so no agent resources are locked until input is submitted.
