# almSemanticVersion

Version following semantic defined by <a href="http://semver.org/">Semantic Versioning</a> document.

[SemanticVersion class](/src/com.santander/alm/pipeline/utils/SemanticVersion.groovy) Helper class for semantic
versioning.

SemanticVersion object contains next methods:

* _SemanticVersion toReleaseVersion():_ if this is a pre-release version returns the corresponding release else
the same version
* _boolean isInDevelopment():_ Returns true if it major version is 0
* _boolean isStable():_ Returns true if it is not a development version
* _boolean isSnapshot():_ Returns true if it is an snapshot version
* _boolean isCompatible(SemanticVersion version):_ Returns true if version is compatible


**`parse`**

Creates the object SemanticVersion @see com.santander.alm.pipeline.utils.SemanticVersion
from a version string (1.0.0, 2.0.0-SNAPSHOT, ...)

* Parameters:
    * _version:_ (Required) Version String
* Returns:
    * SemanticVersion

Example:

```groovy
    steps {
        script {
    	    def sv = almSemanticVersion.parse('1.0.0-SNAPSHOT')
    		println ("sv = " + sv.toString())
    		println "Is dev " + sv.isInDevelopment()
    		println (almSemanticVersion.nextPatch(sv).toString())
    	}
    }
```

**`create(int major, int minor, int patch)`**
Creates the object SemanticVersion from version, release and patch

* Parameters:
    * _major:_ (Required) Version
    * _minor:_ (Required) Release
    * _patch:_ (Required) Patch
* Returns:
    * SemanticVersion

```groovy
    steps {
        script {
    	    def sv = almSemanticVersion.create(1, 0, 0)
    	}
    }
```

**`create(int major, int minor, int patch, String separator, String special)`**:
Creates the object SemanticVersion from version, release and patch

* Parameters:
    * _major:_ (Required) Version
    * _minor:_ (Required) Release
    * _patch:_ (Required) Patch
    * _separator:_ (Required) Special separator
    * _special:_ (Required) Special version ('SNAPSHOT', 'alfa1',..)
* Returns:
    * SemanticVersion

```groovy
    steps {
        script {
    	    def sv = almSemanticVersion.create(1, 0, 0, '-', 'SNAPSHOT')
    	}
    }
```
**`nextMajor(SemanticVersion sm)`**
Returns the next major version of the given semantic version

* Parameters:
    * _sm:_ (Required) SemanticVersion object to get the next version
* Returns:
    * SemanticVersion

```groovy
    steps {
        script {
            def sv = almSemanticVersion.parse('1.0.0-SNAPSHOT')
    	    def next = almSemanticVersion.nextMajor(sv)
    	}
    }
```

**`nextMinor(SemanticVersion sm)`**
Returns the next minor version of the given semantic version

* Parameters:
    * _sm:_ (Required) SemanticVersion object to get the next version
* Returns:
    * SemanticVersion

**`nextPatch(SemanticVersion sm)`**
Returns the next patch version of the given semantic version

* Parameters:
    * _sm:_ (Required) SemanticVersion object to get the next minor version
* Returns:
    * SemanticVersion
