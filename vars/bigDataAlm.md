# bigDataAlm

Clase para la ejecución de una contrucción de bigdata

**`beginBuild`**

Ejecuta una construccion. Si tiene que iniciar el proceso de liberacion de software lo hace. Almacena datos temporales que luego se usaran en el despliegue.

Si la intencion es hacer solo despliegue (sin construccion) verifica si se esta ejecutando desde la rama estable y calcula la version si esta no fue pasada como parametro.

Opcionalmente se le puede pasar como parametro los goals de Maven a ejecutar durante esta fase y parametros opcionales a Maven (-D...)

Tipicamente se ejecuta en la fase build de esta forma:

```groovy
stage ('Build') {
    steps {
        script {
            bigDataAlm.beginBuild()
        } 
     }
}
```

Un ejemplo más completo:
```groovy
stage ('Build') {
    steps {
        script {
            bigDataAlm.beginBuild('clean package','-DpropiedadQueNecesitaMiProyecto=XYZ')
        } 
     }
}
```


**`endBuild`**

Sube el artefacto generado a Nexus. Si se esta en modo release finaliza el proceso de generacion de la version final.

Tipicamente se ejecuta en la fase **Upload to Nexus** de esta forma

```groovy
stage("Upload to Nexus") {
    when {
        allOf {
            anyOf { 
                branch env.MASTER_BRANCH; 
                branch env.INTEGRATION_BRANCH
            }
            not { expression { return params.DEPLOY_ONLY } }
        }
    }
    steps {
        script {
            bigDataAlm.endBuild()
        }
    }
}
```

Opcionalmente se le pueden pasar parametros para que los transmita a Maven:

```groovy
stage("Upload to Nexus") {
    steps {
        script {
            bigDataAlm.endBuild('-Dparametro=XYZ')
        }
    }
}
```

**`getInstaller`**

Devuelve la ruta al instalador. La calcula si no esta previamente almacenada. En el caso de que la construcción sea
de la rama de integración o de release, devuelve la URL de nexus, en otro caso del directorio local de construcción 
(target) y hace stash del fichero para poder recuperarlo posteriormente.

```groovy
stage("Deploy DEV") {
    steps {
        script {
            def deployConf = new ApplicationConfig(this).readCIConfiguration("application.yml")
            deploymentFramework deployConf.edgeNodeDEV, bigDataAlm.getInstaller(), '', 'cbigdataKey', 'cbigdata'
        }
    }
}
```


**NOTA:** Las directivas *when* indican las condiciones para ejecutar esta fase. El ejemplo solo ejecuta esta fase en master y stable únicamente si no se ha ejecutado el job manualmente con el parametro *deployOnly*.

Esto implica que los snapshots de ramas feature **no** suben a Nexus. Si se quiere que los snapshots de una rama feature suban a Nexus modificar las condiciones.

**Note:** This step must be executed in *maven* type agent