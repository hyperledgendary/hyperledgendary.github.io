---
layout: default
title: Java Chaincode Packaging
parent: Good Practice
---

There are many tools to help build Java libraries and applications. Gradle and Maven being the two main tools currently, but not forgetting others such as Ant or the original `javac` and `jar`. Dependency management has also come a long way from Java's origins. 

An impact on this is that Java builds today are more reliant on network access, either to pull the updated tooling or to fetch dependencies. When running in a constrained or air-gapped environment this itself can be challenge.

There are several ways to configure your Java chaincode for building. So you can use the tooling you prefer, and also cater for environment considerations.

## Lifecycle
As shown in the examples, a Java chaincode project (with say Java source, properties files, gradle or maven build files, etc..) is packaged into a `tar.gz` (or `cds` file for v1.4 Fabric)

When a chaincode is installed, approved and committed, the peer will take the 'packaged-resources' provided and create a docker container to 'build' the 'packaged-resources' (this container is emphermal and is not kept). A second runtime docker image is then created with the results of the 'build' and is then 'started'

Two key questions are (1) how does this 'build' work?  (2) how is the code 'started'?

> **Recommendation #0: Are you able to use the external chaincode builders, and chaincode-as-a-service? If so use them, you can then ignore the rest of this document**
> 
> **Recommendation #1: Please use Fabric v2 peers, not v1.4 - they give more flexability in building and starting chaincode**

## How does the build work?

The container image that is used to build the Java Chaincode is `hyperledger/fabric-javaenv`. This docker image is defined in this [dockerfile](https://github.com/hyperledger/fabric-chaincode-java/tree/release-2.2/fabric-chaincode-docker). Note that the Java Version that is used here is **Java11**. This dockerfile is for the open source Fabric - IBP uses it's own image, but follows the open source as to the scripts used.

The peer will take the files you have packaged, and then using the resources in this docker image run the 'build'

### The 'build.sh' script

Take a look at this [build.sh](https://github.com/hyperledger/fabric-chaincode-java/blob/release-2.2/fabric-chaincode-docker/build.sh) script at the current LTS level. 

As per the script, we assume that all the files you packaged are in the `INPUT` directory

1. The filenames and number of any files in `INPUT` that have `.jar` extension are counted. (not a recursive search)
2. A search in the `INPUT` or the `INPUT\src` sub-directory is done to find if there are any `build.gralde` `build.gradle.kts` or `pom.xml` files
   1. if a gradle file is found then will be then used
   2. if NO gradle file is found but a a maven file is found then that will be used 
3. If a gradle build file is present
   1. If in the `INPUT` directory there is the gradle wrapper present (`gradlew`), the build will be started by issuing `./gradlew build shadowJar -x test`
   2. If there is no gradle wrapper, the gradle wrapper that is part of the docker image will be called. `   /root/chaincode-java/gradlew build shadowJar -x test`
   3. It is assumed that either of these builds will produce a `build/libs/chaincode.jar` 'uber-jar'
   4. BUILD SCRIPT ENDS here and copies the `chaincode.jar` to the `OUTPUT`
4. If a maven build fine is present (and no gradle)
   1. If in the `INPUT` directory there is the maven wrapper present (`mvnw`), the build will be started by issuing ` ./mvnw compile package -DskipTests -Dmaven.test.skip=true`
   2. If there is no maven wrapper, the maven wrapper that is part of the docker image will be called. `   /root/chaincode-java/mvnw compile package -DskipTests -Dmaven.test.skip=true`
   3. It is assumed that either of these builds will produce a `target/chaincode.jar ` 'uber-jar'
   4. BUILD SCRIPT ENDS here and copies the `chaincode.jar` to the `OUTPUT`
5. If either step 3 or 4 ended up NOT running, because there where no gradle or maven build files and the number of JARs counted in step 1 is 0, the build script ends here with an error. There is no way to proceed
6. If there are JAR files though, these will be copied to the `OUTPUT`

> **Recommendation: #3 Build you chaincode ahead of time, test and package only the JAR files**

### Implications of the script

This script lets you do several different types of builds
- No builds, just provide JAR files within the chaincode package root directory. Don't include any `build.gradle` or `pom.xml` files. Just one or more JARs that together make up the complete chaincode package.
- If you have both gradle and maven files present, gradle will be used as first choice
- If you provide the gradle or maven wrappers as part of the chaincode package - they will be used. You can therefore use what ever version of the build tools you want.


## Start script

Take a look at this [start](https://github.com/hyperledger/fabric-chaincode-java/blob/release-2.2/fabric-chaincode-docker/start) script at the current LTS level. (Script is bash, but doesn't have the .sh extension, for reasons lost in time)

1. In the case that you supplied a build file, and maven/gradle was used to build the code, then an `.uberjar` file is created to act as a marker.  This means that the file contains everything it needs.

**OR** if you have packaged a single uberjar already, and also packaged a `.uberjar` file.

This is then run `java -jar <filename>.jar` 

Filename will worked out from the sole jar file - if there is more than one, this will exit the script with a failure.


2. If you have supplied more than one JAR, and no `.uberjar` file. All these JARs will be added to a classpath. 
   1. Also the docker image will contain the JAR files for the current LTS version of the chaincode library. These will be *appended* to the classpath.

The JVM will be started with `java -cp ${CHAINCODE_JARS}:${LIB_JARS} org.hyperledger.fabric.contract.ContractRouter`

> **Recommendation #4: Make sure to add a .uberjar file so just your uberjar is run. Otherwise other classes will be added that may conflict**

## Summary

Importants to remember

- Your packaged Java chaincode can contain prebuilt jar file(s). This means that NO network access is required when installatin/approving/committing chaincode. `org.hyperledger.fabric.contract.ContractRouter` must be the main class
- This JAR file can be an 'uber-jar' with everything inside it - remember to add the `.uberjar` file so as to not have a possibly polluted classpath
- You can have multiple JAR files but remember the jars in the `lib` folder will be appended to the classpath
- Check that no build files are included if you choose this route.
- For examles on how to build an uber jar see the examples in [https://github.com/hyperledger/fabric-chaincode-java/tree/release-2.2/examples]()


> **Recomendation #5: Check yourself what's in the JavaEnv docker image.. Run  `docker run -it --entrypoint bash hyperledger/fabric-javaenv:2.2` and look  in the `lib` directory**

- You can supply your own copy of the gradle or maven wrapper to control version of the tool used


