ITDoP
=====

Android new build system: is there a problem when an instrumentTest declares that it depends on a sibling project?

Commit 8b433aa introduces a dependency from an instrumentTest in the App module/project onto code
in a module/project named TestSupport. Immediately, gradle fails to compile the test code:

<pre>
$ ./gradlew clean connectedCheck
:App:clean
:TestSupport:clean
:App:preBuild UP-TO-DATE
:App:preDefaultFlavorDebugBuild UP-TO-DATE
:App:preDefaultFlavorReleaseBuild UP-TO-DATE
:App:prepareComAndroidSupportAppcompatV71900Library
:App:prepareDefaultFlavorDebugDependencies
:App:compileDefaultFlavorDebugAidl
:App:compileDefaultFlavorDebugRenderscript
:App:generateDefaultFlavorDebugBuildConfig
:App:mergeDefaultFlavorDebugAssets
:App:mergeDefaultFlavorDebugResources
:App:processDefaultFlavorDebugManifest
:App:processDefaultFlavorDebugResources
:App:generateDefaultFlavorDebugSources
:App:compileDefaultFlavorDebug
:App:dexDefaultFlavorDebug
:App:processDefaultFlavorDebugJavaRes UP-TO-DATE
:App:validateDebugSigning
:App:packageDefaultFlavorDebug
:App:assembleDefaultFlavorDebug
:App:preDefaultFlavorTestBuild UP-TO-DATE
:App:prepareDefaultFlavorTestDependencies
:App:compileDefaultFlavorTestAidl
:App:compileDefaultFlavorTestRenderscript
:App:processDefaultFlavorTestTestManifest
:App:generateDefaultFlavorTestBuildConfig
:App:mergeDefaultFlavorTestAssets
:App:mergeDefaultFlavorTestResources
:App:processDefaultFlavorTestResources
:App:generateDefaultFlavorTestSources
:App:compileDefaultFlavorTest
/absolute/path/to/ITDoP/App/src/instrumentTest/java/in/sinking/itdop/test/ATest.java:7: cannot find symbol
symbol  : variable InASiblingProject
location: class in.sinking.itdop.test.ATest
        assertTrue(InASiblingProject.returnsTrue());
                   ^
1 error
:App:compileDefaultFlavorTest FAILED
</pre>

It cleaned the TestSupport package, but didn't assemble it. Explicitly asking it to do so makes it work no problem:

<pre>
$ ./gradlew clean :TestSupport:assemble connectedCheck
:App:clean
:TestSupport:clean UP-TO-DATE
:TestSupport:compileJava
:TestSupport:processResources UP-TO-DATE
:TestSupport:classes
:TestSupport:jar
:TestSupport:assemble
:App:preBuild UP-TO-DATE
:App:preDefaultFlavorDebugBuild UP-TO-DATE
:App:preDefaultFlavorReleaseBuild UP-TO-DATE
:App:prepareComAndroidSupportAppcompatV71900Library
:App:prepareDefaultFlavorDebugDependencies
:App:compileDefaultFlavorDebugAidl
:App:compileDefaultFlavorDebugRenderscript
:App:generateDefaultFlavorDebugBuildConfig
:App:mergeDefaultFlavorDebugAssets
:App:mergeDefaultFlavorDebugResources
:App:processDefaultFlavorDebugManifest
:App:processDefaultFlavorDebugResources
:App:generateDefaultFlavorDebugSources
:App:compileDefaultFlavorDebug
:App:dexDefaultFlavorDebug
:App:processDefaultFlavorDebugJavaRes UP-TO-DATE
:App:validateDebugSigning
:App:packageDefaultFlavorDebug
:App:assembleDefaultFlavorDebug
:App:preDefaultFlavorTestBuild UP-TO-DATE
:App:prepareDefaultFlavorTestDependencies
:App:compileDefaultFlavorTestAidl
:App:compileDefaultFlavorTestRenderscript
:App:processDefaultFlavorTestTestManifest
:App:generateDefaultFlavorTestBuildConfig
:App:mergeDefaultFlavorTestAssets
:App:mergeDefaultFlavorTestResources
:App:processDefaultFlavorTestResources
:App:generateDefaultFlavorTestSources
:App:compileDefaultFlavorTest
:App:dexDefaultFlavorTest
:App:processDefaultFlavorTestJavaRes UP-TO-DATE
:App:packageDefaultFlavorTest
:App:assembleDefaultFlavorTest
:App:connectedInstrumentTestDefaultFlavorDebug
:App:connectedInstrumentTest
:App:connectedCheck

BUILD SUCCESSFUL
</pre>

Should I have expected this to work, or have I just misunderstood something?

Workaround
----------

Adding the following to App/build.gradle seems to patch things up, but maybe there's a better way?

    afterEvaluate { project ->
        project.tasks.getByName('compileDefaultFlavorTest').dependsOn ':TestSupport:assemble'
    }