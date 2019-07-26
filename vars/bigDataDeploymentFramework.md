# bigDataDeploymentFramework

Despliega un proyecto construido bajo lo definido por [Deployment Framework](https://gitlab.alm.gsnetcloud.corp/ci-bigdata/deployment-framework)

Consultar la documentacion de deployment framework aqui: https://gitlab.alm.gsnetcloud.corp/ci-bigdata/deployment-framework/wikis/home

Acepta los siguientes parametros:

Parametro | Obligatorio | Descripcion 
--- | :---: | ---
**edgeNode** | Si | Mapa clave valor con la configuracion necesaria para el despliegue. A continuacion se muestra la estructura con las claves necesarias.
**installerLocation** | Si | Ruta (path o url) al instalador. Se puede obtener ejecutando `bigDataAlm.getInstaller()` 
**deploymentFrameworkParams** | No | Parametros opcionales a pasar al instalador. Por ejemplo en instalaciones Supra hay que pasar `-Dsupra.edgenode.installMode=primary` al primer nodo frontera y `-Dsupra.edgenode.installMode=secondary` al resto
**credentialsid** | No | Si se proporcionan credenciales (id de como estan almacenadas en Jenkins) se usaran en el despliegue y no se preguntara al usuario 
**username** | No | Si se proporcionan credenciales en el parametro anterior debera proporcionarse el nombre de usuario asociado a las mismas 
**showPromptToContinueDeployment** | No |  El valor por defecto (false) instala sin preguntar con las credenciales proporcionadas. Si se indica true espera a que un usuario apruebe el despliegue antes de continuar.

El mapa con los parametros para el despliegue se pueden leer de un fichero application.yml con la siguiente estructura:

```yaml
ci:
  nodeName:
      gitCredentialsId: <id de las credenciales para acceder al git de configuracion>
      confGitUrl : <url al git de configuracion>
      host: <nodo donde hacer el despliegue>
      keytab: <OPCIONAL ruta relativa al home del usuario o absoluta al fichero keytab para obtener ticket Kerberos si no se obtiene automaticamente al iniciar sesion>
      antCommand: <OPCIONAL ruta al ejecutable de ant por si no estan creadas en el entorno las variables apropiadas >
      workDirPrefix: <prefijo a la ruta temporal a utilizar>
      environmentFile : <nombre del fichero con las propiedades de entorno a reemplazar durante la instalacion>
 ```
 
 Por ejemplo :
 
 ```yaml
ci:
  edgeNodeDEV:
      gitCredentialsId: 41d889a1-823f-4fff-b779-b1df794d1e62
      confGitUrl : ssh://git@gitlab.alm.gsnetcloud.corp:2220/ci-bigdata/deployment-properties.git
      host: cnsalbsrvcl22.lvtc.gsnet.corp
      keytab: cbigdata.keytab
      workDirPrefix: /tmp/install/output-manager-installer
      environmentFile : environment_DEV.properties
  edgeNodeCER:
      gitCredentialsId: 41d889a1-823f-4fff-b779-b1df794d1e62
      confGitUrl : ssh://git@gitlab.alm.gsnetcloud.corp:2220/ci-bigdata/deployment-properties.git
      host: cnsalbsrvcl22.lvtc.gsnet.corp
      workDirPrefix: /tmp/install/output-manager-installer
      environmentFile : environment_DEV.properties
  edgeNodePRE:
      gitCredentialsId: 41d889a1-823f-4fff-b779-b1df794d1e62
      confGitUrl : ssh://git@gitconf.santander.corp:2220/ImpEs/<proyecto>_PRE.git
      host: cnsalbsrvcl22.lvtc.gsnet.corp
      workDirPrefix: /tmp/install/output-manager-installer
      antCommand: /opt/ant-installation-folder/bin/ant
      environmentFile : environment_PRE.properties
 ```
 
 Se invoca por ejemplo de esta forma:
 
 ```groovy
deploymentFramework deployConf.edgeNodePRE1, bigDataAlm.getInstaller()
```

Para obtener `deployConf` se puede ejecutar lo siguiente:

```groovy
deployConf = new ApplicationConfig(this).readCIConfiguration("application.yml")
```
Se recomienda generar deployConf justo antes de ejecutar el despliegue al primer entorno.

Tambien se puede generar el mapa directamente en el Jenkinsfile:
```groovy
deployConf = [ 'gitCredentialsId' : '41d889a1-823f-4fff-b779-b1df794d1e62', 
      'confGitUrl' : 'ssh://git@gitlab.alm.gsnetcloud.corp:2220/ci-bigdata/deployment-properties.git', 
      'host' : 'cnsalbsrvcl22.lvtc.gsnet.corp', 
      'workDirPrefix' : '/tmp/install/e2a', 
      'environmentFile' : 'e2a_environment_DEV.properties', 
      'antCommand' : '/etc/apache-ant-1.9.7/bin/ant'] 
```




### Despliegues Supra a más de un nodo frontera

Para hacer despliegues Supra se deberá invocar al deployment framework de la siguiente manera:

 ```groovy
deploymentFramework deployConf.edgeNodePRE1, bigDataAlm.getInstaller(), '-Dsupra.edgenode.installMode=primary'
deploymentFramework deployConf.edgeNodePRE2, bigDataAlm.getInstaller(), '-Dsupra.edgenode.installMode=secondary'
deploymentFramework deployConf.edgeNodePRE3, bigDataAlm.getInstaller(), '-Dsupra.edgenode.installMode=secondary'
```
En el nodo primario se actualiza la bbdd Postgress y se copian ficheros a HDFS. En los secundarios solo se copian los ficheros necesarios del nodo frontera.
