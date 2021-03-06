[//]: # " "
[//]: # " Copyright (c) 2013, 2021 Oracle and/or its affiliates. All rights reserved. "
[//]: # " "
[//]: # " This program and the accompanying materials are made available under the "
[//]: # " terms of the Eclipse Public License v. 2.0, which is available at "
[//]: # " http://www.eclipse.org/legal/epl-2.0. "
[//]: # " "
[//]: # " This Source Code may also be made available under the following Secondary "
[//]: # " Licenses when the conditions for such availability set forth in the "
[//]: # " Eclipse Public License v. 2.0 are satisfied: GNU General Public License, "
[//]: # " version 2 with the GNU Classpath Exception, which is available at "
[//]: # " https://www.gnu.org/software/classpath/license.html. "
[//]: # " "
[//]: # " SPDX-License-Identifier: EPL-2.0 OR GPL-2.0 WITH Classpath-exception-2.0 "
[//]: # " "

* TOC
{:toc}

# Inhabitant Generators

There are two ways to generate hk2 metadata (called inhabitant files) that can be
used by the runtime system to find hk2 services without having to classload those
services.  The first is the <b>HK2 Metadata Generator</b>, which works with the javac build
tool, and the second is the <b>HK2 Inhabitant Generator</b> which can be used at build time
with any build system or even post build time with JARs or can be embedded in other
tools.

The HK2 Metadata Generator is easy to use, as all it requires is to be on the
javac classpath.  The HK2 Inhabitant Generator is also easy to use
but requires the user to specify lines in the build system files (besides just
dependencies).  Users can choose whichever tool works for them in their
build environment.

## HK2 Metadata Generator

The HK2 Metadata Generator will generate hk2 inhabitant files during the compilation
of your java files.  It is a JSR-269 annotation processor that handles the [@Service][service]
annotation.  The only requirement for using it is to put the javax.inject, hk2-utils, hk2-api and
hk2-metadata-generator libraries in the classpath of the javac process.

In this example we use the HK2 Metadata Generator in a Maven based build system:

```xml
  <dependency>
      <groupId>org.glassfish.hk2</groupId>
      <artifactId>hk2-metadata-generator</artifactId>
  </dependency>
```

Since Maven uses transitive dependencies this is all you need to add as a dependency during
build.

In the following example we use the HK2 Metadata Generator in a gradle based build system:

```java
dependencies {
  compile group: 'javax.inject', name: 'javax.inject', version: '1.1'
  compile group: 'org.glassfish.hk2', name: 'hk2-utils', version: '2.4.0-b14'
  compile group: 'org.glassfish.hk2', name: 'hk2-api', version: '2.4.0-b14'
  compile group: 'org.glassfish.hk2', name: 'hk2-metadata-generator', version: '2.4.0-b14'
  
  testCompile group: 'javax.inject', name: 'javax.inject', version: '1.1'
  testCompile group: 'org.glassfish.hk2', name: 'hk2-utils', version: '2.4.0-b14'
  testCompile group: 'org.glassfish.hk2', name: 'hk2-api', version: '2.4.0-b14'
  testCompile group: 'org.glassfish.hk2', name: 'hk2-metadata-generator', version: '2.4.0-b14'
}
```

### HK2 Metadata Generator Options

By default the HK2 Metadata Generator places the output file in META-INF/hk2-locator/default. However
this behavior can be modified by setting the option <i>org.glassfish.hk2.metadata.location</i> to
the desired location.  This is done with the javac compiler using the -A option.  In gradle
this looks something like this:

```java
compileJava {
  options.compilerArgs << '-Aorg.glassfish.hk2.metadata.location=META-INF/hk2-locator/acme'
}
```

## HK2 Inhabitant Generator

The HK2 Inhabitant Generator is a utility that will generate inhabitants file during the
build of your JAR file.  It works by analyzing the classes that have been built by javac and
then creating the file **META-INF/hk2-locator/default** (by default) in your JAR file that has
information in it about all of the classes that you have marked with [@Service][service] or
[@Contract][contract].

The HK2 Inhabitatants Generator can be used as a standalone command-line tool, or it can
be embedded in any Java program.  It can also be used in a Maven build.  An Eclipse build
and an ant task are also planned.  Here are the ways that the HK2 Inhabitants Generator can
be used:

### Command Line Tool

The HK2 Inhabitants Genertor can be run from the command line like this:
 
```java
java org.jvnet.hk2.generator.HabitatGenerator
```

By default the [HabibatGenerator][habitatgenerator] will attempt to analyze the first element of the classpath
and replace that element (if it is a JAR) with a new JAR that has an inhabitants file in
it describing all of the classes marked with [@Service][service].  If the first element of the classpath
is a directory it will attempt to create a new inhabitants file in that directory describing
all of the classes marked with [@Service][service].

You can modify this behavior by using command line options.
Here is the usage statement for [HabibatGenerator][habitatgenerator]:
 
```
java org.jvnet.hk2.generator.HabitatGenerator
  [--file jarFileOrDirectory]
  [--outjar jarFile]
  [--locator locatorName]
  [--verbose]
```

The --file option allows the user to pick a directory or JAR file to analyze for classes marked
with @Service.

The --outjar option allows the user to pick the output JAR file that the generator should create
with the inhabitants file in it.

The --locator option allows the user to name the locator that these services should go into.  This
value is \"default\" by default.

The --verbose option make the generator print extra information as it does its work.

This command line utility will call **System.exit** when it is done with a 0 code if it was able
to work properly and a non-zero value if it failed in some way.
 
### Embedded Usage

The class [org.jvnet.hk2.generator.HabitatGenerator][habitatgenerator] has a static method on called embeddedMain.
The embeddedMain takes the typical argv[] array of parameters and so has the same behavior
as the command line usage.  The biggest difference is that this method returns an
int as the return code, either 0 for success or non-zero for failure and does not call
System.exit().  See the [javadoc][habitatgenerator] for more information.
 
Using embeddedMain is useful if you want to build your own build tools that generate inhabitants
files for your own IDE or other build environment.
 
### Using Maven

The HabitatGenerator is also available as a Maven plugin.  It has a single goal, called
generateInhabitants that is run in the process-classes phase by default.  Using this plugin
in your build will cause inhabitants files to be generated in your output directory.

The following example plugin snippet from a pom.xml will run the InhabitantsGenerator in
both the main tree and in the test tree, in case you would like your test sources to also
be analyzed for classes marked with [@Service][service].

```xml
    <plugin>
      <groupId>org.glassfish.hk2</groupId>
      <artifactId>hk2-inhabitant-generator</artifactId>
      <version>2.5.0-b36</version>
      <executions>
        <execution>
          <goals>
            <goal>generate-inhabitants</goal>
          </goals>
        </execution>
      </executions>
    </plugin>
```

 The plugin has the following configuration options:
 
+ outputDirectory (the place the output file will go, defaults to $\{project.build.outputDirectory\})
+ testOutputDirectory (the place the output will go if test is true, defaults to $\{project.build.testOutputDirectory\})
+ verbose (true or false)
+ test Set to true if this execution should be for the tests rather than main
+ locator The name of the locator file (which is \"default\" by default)
+ noswap (true or false) if set to true the generator will overwrite files in place which is riskier but faster
  
### Ant Task

The inhabitant generator can also be used as an ant task.  The ant task is org.jvnet.hk2.generator.maven.ant.HK2InhabitantGeneratorTask.
Below find an example ant file that uses the task:
  
```xml
<project name="HK2 Ant Build File" default="build" basedir=".">
  <!-- set global properties for this build -->
  <property name="src" location="src"/>
  <property name="build" location="target/classes"/>

  <taskdef name="hk2-inhabitant-generator"
           classname="org.jvnet.hk2.generator.ant.HK2InhabitantGeneratorTask"/>

  <target name="compile" >
    <!-- Compile the java code from ${src} into ${build} -->
    <javac srcdir="${src}" destdir="${build}"/>
    <hk2-inhabitant-generator targetDirectory="${build}"/>
  </target>
</project>
```

The thing to note in the example above is that the hk2-inhabitant-generator must run after the classes are built, as the hk2-inhabitant-generator
inspects the class files.

The ant plugin has the following options:
 
+ targetDirectory (the directory to find the classes, defaults to target/classes)
+ outputDirectory (the place the output file will go, defaults to target/classes)
+ verbose (true or false)
+ locator The name of the locator file (which is \"default\" by default)
+ noswap (true or false) if set to true the generator will overwrite files in place which is riskier but faster

## Stub Generation

The HK2 metadata generator can also generate implementation classes based on abstract classes.  This is useful
when testing, as it is often the case in tests that the user would like to replace some service with one
that does nothing or does only a few special things during the test.  This is done by putting the [Stub][stub]
annotation on an abstract class.  Any abstract methods NOT implemented by the class will have dummy implementations
generated in a java file that also has an [Service][service] annotation.  This will then become an service in
the Singleton scope, which should show up when using most hk2 initialization methodologies.  The benefits of
using the [Stub][stub] annotation include:

+ The service need not be completely implemented.  Only those methods that need to return specific data
needed by the test need to be implemented
+ The [Rank][rank] annotation will work when placed on the abstract class, so the stub can be given higher
priority than the replacement class
+ Using [Stub][stub] means that this abstract class may not need to be updated if the underlying interface
or class has an added method
+ It makes it very easy to write very simple test replacement classes

In this simple example there is an interface with a large number of methods:

```java
@Contract
public interface ManyMethodsService {
  public void methodA();
  public void methodB();
  public int methodFoo();
  public String methodBar(String input);
  // And so on
}
```

Somewhere in your build there is an implementation of ManyMethodsService.  It may or may not look like this:

```java
@Service
public class ManyMethodsServiceImpl implements ManyMethodsService {
  // Here we really implement the interface, possibly doing JDBC or other possibly heavy operations
}
```

However, in the test environment there is only one or two methods that the test touches.  In this case
we can very easily use the [Stub][stub] annotation to tell the hk2-metadata-generator to generate an
implementation that fills in the missing methods.  This test version of the service might be
implemented like this:

```java
@Stub @Rank(1)
public abstract class TestManyMethodsService implements ManyMethodsService {
  // Only implement the methods that the test actually touches, and return
  // hard-coded data
  public int methodFoo() {
    return 13;
  }
  
  public String methodBar(String input) {
    return input;
  }
  
  // Do not implement the other methods in the interface
}
```

If the hk2-metadata-generator is in the classpath of the test build then it will see the [Stub][stub]
annotation and will generate a java file with the unimplemented methods implemented, returning
either null or 0 or "a" along with an added [Service][service] annotation.  Since the [Rank][rank]
annotation is set to 1 in the abstract class (TestManyMethodsService) it will be used in favor of the true
implementation (ManyMethodsServiceImpl).

If there is a [@Named][named] qualifier on the class annotated with [Stub][stub] that qualifier will also be
copied to the resulting implementation class.  If [@Named][named] has no value associated with it the value
used will be the value of the class with [Stub][stub] on it.  For example this class:

```java
@Stub @Named
public abstract class AliceService implements SomeInterface {
}
```

will end up having the name \"AliceService\", while this class:

```java
@Stub @Named("Bob")
public abstract class AliceService implements SomeInterface {
}
```

will end up having the name \"Bob\".  Only the [@Named][named] qualifier is treated
specially in this way.

The [Stub][stub] annotation can also generate implementations where rather than returning null or zero values
it throws UnsupportedOperationException (with the name of the method called in the message).  This can
be done by setting the value field of the [Stub][stub] to EXCEPTIONS, like this:

```java
@Stub(Stub.Type.EXCEPTIONS)
public abstract class AliceService implements SomeInterface {
  // All generated methods will throw UnsupportedOperationException
}
```

This can be useful when writing tests to ensure that the code does not inadvertently use one of the
methods of the stub's implementation.

[service]: apidocs/org/jvnet/hk2/annotations/Service.html
[contract]: apidocs/org/jvnet/hk2/annotations/Contract.html
[habitatgenerator]: apidocs/org/jvnet/hk2/generator/HabitatGenerator.html
[stub]: apidocs/org/jvnet/hk2/utilities/Stub.html
[rank]: apidocs/org/glassfish/hk2/api/Rank.html
[named]: https://jakarta.ee/specifications/platform/8/apidocs/javax/inject/Named.html
