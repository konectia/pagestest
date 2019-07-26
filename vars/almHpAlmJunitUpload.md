### [almHpAlmJunitUpload](/vars/almHpAlmJunitUpload.groovy)

 Upload in HP_ALM the test results.

**Parameters:**
- *almServerName: HP Alm Server name in Jenkins
- *credentialsId: Jenkins credentials to connect with HP Alm
- *almDomain: HP Alm domain
- *almProject: HP Alm project
- *testingFramework: JUnit, NUnit, TestNG
- *testingTool: Selenium, SoapUI, Watir, Sahi, LeanFT
- *almTestFolder: Test case folder in HP Alm 
- *testingResultFile: Path where test result are stored
- *jenkinsServerUrl: The HTTP URL of the Jenkins Server, form example, http://myjenkinsserver.test.com:8080
- *testSetFields: Custom files for Test Set
- *testCaseFields: Custom files for Test Cases

**NOTE:** testingFramework and testingTool are optional parameters that can be filled in with null.
**NOTE:** Micro Focus Application Automation Tools 5.4.3.1 plugin needed

** HP_ALM on alm1.produban.gs.corp
```groovy

            post {
                always {
                
                    echo "upload junit test results to HP ALM..."
                    almHpAlmJunitUpload (
                        almServerName: "HP ALM Isban",
                        credentialsId: "xIS08672-passw",
                        almDomain: "ISB_GSW",
                        almProject: "HAR",
                        testingFramework: "JUnit",
                        testingTool: null,
                        almTestFolder: "TestPostman_v2\\V01R00",
                        almTestSetFolder: "TestPostman_v2\\V01R00\\Fix001",
                        testingResultFile: 'newman/*.xml',
                        jenkinsServerUrl: "http://oclubunc008.isbcloud.isban.corp:8080",
                        testSetFields: [
                            "user-05": "DES", // Entorno
                            "user-08": "14", // Ciclo
                            "user-09": "V01R00F000", // Aplicacion V.R.
                            "user-template-02": "Si", // Desarrollo Externo
                            "user-13": "Allfunds Bank", // Cliente
                            "user-19": "Proyecto Correctivo", // Tipo de Proyecto
                            "user-20": "9471234", // Codigo de Proyecto
                            "user-15": "QA_Interno", // Grupo
                            "user-17": "252525" // Expediente
                        ],
                        testCaseFields: [
                            "user-02": "End-To-End", // Tipo Prueba
                            "user-22": "SAPA", // Aplicacion
                            "user-21": "Intranet", // Canales Afectados
                            "user-39": "Baja", // Dificultad
                            "status": "Finalizado" // Estado
                        ]
                    )

                }
            }
    
```

** HP_ALM on alm3.produban.gs.corp
```groovy
            post {
                always {
                
                    echo "upload junit test results to HP ALM..."
                    almHpAlmJunitUpload (
                        almServerName: "HP ALM Produban",
                        credentialsId: "xIS08672-passw",
                        almDomain: "QAF_GTS_CORPORACION",
                        almProject: "WORKDAY_HQ",
                        testingFramework: "JUnit",
                        testingTool: null,
                        almTestFolder: "TestALM\\V01R00",
                        almTestSetFolder: "TestALM\\V01R00\\Fix001",
                        testingResultFile: 'newman/*.xml',
                        jenkinsServerUrl: "http://oclubunc008.isbcloud.isban.corp:8080",
                        testSetFields: [ 
                            "user-05" : "Development", // Environment
                            "user-09" : "Default", // SW Delivery
                            "user-08" : "2", // Test Cycle
                            "user-07" : "No", // Delegate
                            "user-06" : "default" // QC Application
                            "user-21" : "1- FUNCTIONAL CERTIFICATION" // Phase
                        ],
                        testCaseFields: [
                            'user-22': 'No' // TC Basic
                        ]
                    )
                }
            }                

```
