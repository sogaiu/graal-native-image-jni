# graal-native-image-jni

### Aim

Try and build the smallest possible JNI example to test GraalVM's native-image JNI support.

### Result

Success.

```
$ ./helloworld
Hello world; this is C talking!
```

### Insight

In order for native-image to successfuly load a c library to execute, it must run the `System.loadLibrary()` call at runtime, not at build time.

### Method 1: Put loadLibrary in the execution path

This is the version we have done. By putting loadLibrary inside the `main` method, the library is loaded at run time. With this setup we can compile with `--initialize-at-build-time` and everything will work.

### Method 2: Put loadLibrary in static class initializer and use --initialize-at-run-time

Sometimes you don't have control over where you call loadLibrary from. Often existing code places it in the slasses static initializer block. In this case the library is loaded at build time, but then when the final artifact is run, the linked code cannot be found and the programme crashes with a `java.lang.UnsatisfiedLinkError` exception.

When you place the loadLibrary call within a static block of a class, you must specify to `native-image` that your class should be initialized at runtime.

## Requirements

 * Linux
 * GraalVM CE 19.3.x with native-image tool installed
 * Working GNU C compiler

## Overview

`HelloWorld.java` contains HelloWorld class, that calls the native code in `HelloWorld.c` to print output.

`HelloWorld.c` compiles into `libHelloWorld.so`

`HelloWorld.class` is built into a jar with a simple manifest.

## Preparation

Ensure `GRAALVM_HOME` and `PATH` are appropriately set.

Note that putting `GRAALVM_HOME/bin` on `PATH` appropriately will lead to `javac` and `jar` being used from Graal.

## Build and run a JNI jar

```
$ make run-jar
javac src/HelloWorld.java
cd src && jar cfm ../HelloWorld.jar manifest.txt HelloWorld.class
cd src && javac -h . HelloWorld.java
gcc -shared -Wall -Werror -I$GRAALVM_HOME/include -I$GRAALVM_HOME/include/linux -o libHelloWorld.so -fPIC src/HelloWorld.c
LD_LIBRARY_PATH=./ java -jar HelloWorld.jar
Hello world; this is C talking!
```

Note: output has been slightly modified for paths.

## Build and run a native image

(you can specify a custom GRAALVM path with `make run-native GRAALVM_HOME=/path/to/my/graalvm`)

```
$ make run-native
cd src && jar cfm ../HelloWorld.jar manifest.txt HelloWorld.class
$GRAALVM_HOME/bin/native-image \
	-jar HelloWorld.jar \
	-H:Name=helloworld \
	-H:+ReportExceptionStackTraces \
	-H:ConfigurationFileDirectories=config-dir \
	--initialize-at-build-time \
	--verbose \
	--no-fallback \
	--no-server \
	"-J-Xmx1g" \
	-H:+TraceClassInitialization -H:+PrintClassInitialization
Executing [
$GRAALVM_HOME/jre/bin/java \
-XX:+UnlockExperimentalVMOptions \
-XX:+EnableJVMCI \
-Dtruffle.TrustAllTruffleRuntimeProviders=true \
-Dtruffle.TruffleRuntime=com.oracle.truffle.api.impl.DefaultTruffleRuntime \
-Dgraalvm.ForcePolyglotInvalid=true \
-Dgraalvm.locatorDisabled=true \
-d64 \
-XX:-UseJVMCIClassLoader \
-XX:+UseJVMCINativeLibrary \
-Xss10m \
-Xms1g \
-Xmx14g \
-Duser.country=US \
-Duser.language=en \
-Dorg.graalvm.version=19.3.0.2 \
-Dorg.graalvm.config=CE \
-Dcom.oracle.graalvm.isaot=true \
-Djvmci.class.path.append=$GRAALVM_HOME/jre/lib/jvmci/graal.jar \
-javaagent:$GRAALVM_HOME/jre/lib/svm/builder/svm.jar=traceInitialization \
-Djdk.internal.lambda.disableEagerInitialization=true \
-Djdk.internal.lambda.eagerlyInitialize=false \
-Djava.lang.invoke.InnerClassLambdaMetafactory.initializeLambdas=false \
-Xmx1g \
-Xbootclasspath/a:$GRAALVM_HOME/jre/lib/boot/graal-sdk.jar:$GRAALVM_HOME/jre/lib/boot/graaljs-scriptengine.jar \
-cp \
$GRAALVM_HOME/jre/lib/svm/builder/svm-llvm.jar:$GRAALVM_HOME/jre/lib/svm/builder/graal-llvm.jar:$GRAALVM_HOME/jre/lib/svm/builder/llvm-platform-specific-shadowed.jar:$GRAALVM_HOME/jre/lib/svm/builder/svm.jar:$GRAALVM_HOME/jre/lib/svm/builder/objectfile.jar:$GRAALVM_HOME/jre/lib/svm/builder/pointsto.jar:$GRAALVM_HOME/jre/lib/svm/builder/llvm-wrapper-shadowed.jar:$GRAALVM_HOME/jre/lib/svm/builder/javacpp-shadowed.jar:$GRAALVM_HOME/jre/lib/jvmci/jvmci-api.jar:$GRAALVM_HOME/jre/lib/jvmci/graal.jar:$GRAALVM_HOME/jre/lib/jvmci/jvmci-hotspot.jar:$GRAALVM_HOME/jre/lib/jvmci/graal-management.jar \
com.oracle.svm.hosted.NativeImageGeneratorRunner \
-watchpid \
7247 \
-imagecp \
$GRAALVM_HOME/jre/lib/boot/graal-sdk.jar:$GRAALVM_HOME/jre/lib/boot/graaljs-scriptengine.jar:$GRAALVM_HOME/jre/lib/svm/builder/svm-llvm.jar:$GRAALVM_HOME/jre/lib/svm/builder/graal-llvm.jar:$GRAALVM_HOME/jre/lib/svm/builder/llvm-platform-specific-shadowed.jar:$GRAALVM_HOME/jre/lib/svm/builder/svm.jar:$GRAALVM_HOME/jre/lib/svm/builder/objectfile.jar:$GRAALVM_HOME/jre/lib/svm/builder/pointsto.jar:$GRAALVM_HOME/jre/lib/svm/builder/llvm-wrapper-shadowed.jar:$GRAALVM_HOME/jre/lib/svm/builder/javacpp-shadowed.jar:$GRAALVM_HOME/jre/lib/jvmci/jvmci-api.jar:$GRAALVM_HOME/jre/lib/jvmci/graal.jar:$GRAALVM_HOME/jre/lib/jvmci/jvmci-hotspot.jar:$GRAALVM_HOME/jre/lib/jvmci/graal-management.jar:$GRAALVM_HOME/jre/lib/svm/library-support.jar:...graal-native-image-jni/HelloWorld.jar \
-H:Path=...graal-native-image-jni \
-H:Class=HelloWorld \
-H:+ReportExceptionStackTraces \
-H:ConfigurationFileDirectories=config-dir \
-H:ClassInitialization=:build_time \
-H:FallbackThreshold=0 \
-H:+TraceClassInitialization \
-H:+PrintClassInitialization \
-H:CLibraryPath=$GRAALVM_HOME/jre/lib/svm/clibraries/linux-amd64 \
-H:Name=helloworld
]
[helloworld:7271]    classlist:   1,308.37 ms
[helloworld:7271]        (cap):     775.78 ms
[helloworld:7271]        setup:   1,597.48 ms
Printing initializer configuration to ...graal-native-image-jni/reports/initializer_configuration_20200120_200440.txt
[helloworld:7271]   (typeflow):   2,915.15 ms
[helloworld:7271]    (objects):   2,904.21 ms
[helloworld:7271]   (features):     147.05 ms
[helloworld:7271]     analysis:   6,169.77 ms
Printing initializer dependencies to ...graal-native-image-jni/reports/initializer_dependencies_20200120_200447.dot
Printing 0 classes that are considered as safe for build-time initialization to ...graal-native-image-jni/reports/safe_classes_20200120_200447.txt
Printing 1691 classes of type BUILD_TIME to ...graal-native-image-jni/reports/build_time_classes_20200120_200447.txt
Printing 43 classes of type RERUN to ...graal-native-image-jni/reports/rerun_classes_20200120_200447.txt
Printing 0 classes of type RUN_TIME to ...graal-native-image-jni/reports/run_time_classes_20200120_200447.txt
[helloworld:7271]     (clinit):     103.39 ms
[helloworld:7271]     universe:     318.91 ms
[helloworld:7271]      (parse):     315.21 ms
[helloworld:7271]     (inline):     838.22 ms
[helloworld:7271]    (compile):   2,606.59 ms
[helloworld:7271]      compile:   3,967.77 ms
[helloworld:7271]        image:     311.69 ms
[helloworld:7271]        write:      84.33 ms
[helloworld:7271]      [total]:  13,979.30 ms
LD_LIBRARY_PATH=./ ./helloworld
Hello world; this is C talking!
```

Note: output has been slightly modified for paths.
