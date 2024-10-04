### Spring Boot Native with Redisson

This project explains the problem that I am facing when trying to generate a native image using GraalVM of a Spring Boot application that uses Redisson and the DataDog tracing agent.

### Problem

When I try to generate a native image of a Spring Boot application that uses Redisson, I get the following error:

```
[2/8] Performing analysis...  []                                                                         (5.1s @ 0.87GB)
   5,369 (61.17%) of  8,777 types reachable
   5,134 (43.16%) of 11,895 fields reachable
  13,358 (24.11%) of 55,394 methods reachable
   3,255 types, 1,125 fields, and 12,307 methods registered for reflection

Fatal error: com.oracle.svm.core.util.VMError$HostedError: guarantee failed
	at org.graalvm.nativeimage.builder/com.oracle.svm.core.util.VMError.shouldNotReachHere(VMError.java:72)
	at org.graalvm.nativeimage.builder/com.oracle.svm.core.util.VMError.guarantee(VMError.java:86)
	at org.graalvm.nativeimage.builder/com.oracle.svm.core.hub.ClassForNameSupport.registerClass(ClassForNameSupport.java:63)
	at org.graalvm.nativeimage.builder/com.oracle.svm.hosted.reflect.ReflectionDataBuilder.registerTypesForGenericSignature(ReflectionDataBuilder.java:726)
	at org.graalvm.nativeimage.builder/com.oracle.svm.hosted.reflect.ReflectionDataBuilder.registerTypesForGenericSignature(ReflectionDataBuilder.java:689)
	at org.graalvm.nativeimage.builder/com.oracle.svm.hosted.reflect.ReflectionDataBuilder.registerTypesForClass(ReflectionDataBuilder.java:563)
	at org.graalvm.nativeimage.builder/com.oracle.svm.hosted.reflect.ReflectionDataBuilder.registerHeapDynamicHub(ReflectionDataBuilder.java:969)
	at org.graalvm.nativeimage.builder/com.oracle.svm.hosted.heap.SVMImageHeapScanner.onObjectReachable(SVMImageHeapScanner.java:155)
	at org.graalvm.nativeimage.pointsto/com.oracle.graal.pointsto.heap.ImageHeapScanner.lambda$createImageHeapObject$5(ImageHeapScanner.java:396)
	at org.graalvm.nativeimage.pointsto/com.oracle.graal.pointsto.heap.ImageHeapScanner.lambda$postTask$14(ImageHeapScanner.java:759)
	at org.graalvm.nativeimage.pointsto/com.oracle.graal.pointsto.util.CompletionExecutor.executeCommand(CompletionExecutor.java:187)
	at org.graalvm.nativeimage.pointsto/com.oracle.graal.pointsto.util.CompletionExecutor.lambda$executeService$0(CompletionExecutor.java:171)
	at java.base/java.util.concurrent.ForkJoinTask$RunnableExecuteAction.exec(ForkJoinTask.java:1395)
	at java.base/java.util.concurrent.ForkJoinTask.doExec(ForkJoinTask.java:373)
	at java.base/java.util.concurrent.ForkJoinPool$WorkQueue.topLevelExec(ForkJoinPool.java:1182)
	at java.base/java.util.concurrent.ForkJoinPool.scan(ForkJoinPool.java:1655)
	at java.base/java.util.concurrent.ForkJoinPool.runWorker(ForkJoinPool.java:1622)
	at java.base/java.util.concurrent.ForkJoinWorkerThread.run(ForkJoinWorkerThread.java:165)
```

I am generating the native image using the following command `./mvnw native:compile -Pnative` as stated in the official spring boot doc (https://docs.spring.io/spring-boot/how-to/native-image/developing-your-first-application.html#howto.native-image.developing-your-first-application.native-build-tools).
My java version is 
```
❯ java --version
java 17.0.12 2024-07-16 LTS
Java(TM) SE Runtime Environment Oracle GraalVM 17.0.12+8.1 (build 17.0.12+8-LTS-jvmci-23.0-b41)
Java HotSpot(TM) 64-Bit Server VM Oracle GraalVM 17.0.12+8.1 (build 17.0.12+8-LTS-jvmci-23.0-b41, mixed mode, sharing)
```

Note that if I remove the Redisson dependency from the project, the native image is generated successfully. The same happens when I generate the native image without the DataDog tracing agent.

### Attempt to solve the problem
I have tried to generate the hints to pass to the `native-image` tool by running the jar with the tracing agent as explained in https://www.graalvm.org/latest/reference-manual/native-image/metadata/AutomaticMetadataCollection/.
Moreover, the build is successful when I run the build process by using the Liberica NIK tool (https://bell-sw.com/liberica-native-image-kit/).