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

# Getting started

## Maven Build

The best way to use HK2 in your builds is to add the following dependency in your maven build:

```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.acme</groupId>
    <artifactId>myModule</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
      <dependency>
        <groupId>org.glassfish.hk2</groupId>
        <artifactId>hk2</artifactId>
        <version>2.5.0-b36</version>
      </dependency>
    </dependencies>
</project>
```

The org.glassfish.hk2:hk2 dependency has a dependency on all of the HK2 jars.
However, this may be more than you want, since it includes configuration, 
run-level services and some osgi support that your application may not need.
If instead you wanted the absolute minimum working profile for hk2 you would instead have your project look like this:

```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.acme</groupId>
    <artifactId>myModule</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
      <dependency>
        <groupId>org.glassfish.hk2</groupId>
        <artifactId>hk2-locator</artifactId>
        <version>2.5.0-b36</version>
      </dependency>
    </dependencies>
</project>
```

The hk2-locator project contains the implementation of the hk2 API, with
no other bells and whistles.  In particular, the ability to automatically
detect services is not available, and so all HK2 objects must be added
programmatically and gotten with the HK2 API.  However, the above is perfect
for small projects that want to play with the HK2 API to see how it works.

## Automatic Service Population

In order for HK2 to automatically find services at runtime it can read files called inhabitant files.
 These are usually placed in your JAR file at location META-INF/hk2-locator.
 Normally the file is named default.
 (You can however use a different file name or location(s) by using more specific API).
 HK2 has a tool for automatically creating these files based on class files annotated with [@Service][service].
 There is also a simple API for creating and populating a [ServiceLocator][servicelocator] with services found in these files.

In order to have your Maven build generate the META-INF files that hk2 reads in order to populate a [ServiceLocator][servicelocator] 
use the [hk2-inhabitant-generator][inhabitant-generator].
This tool can be used from the command line, or it can be put into your maven or ant builds.

In order to have your program automatically load the files generated with the [hk2-inhabitant-generator][inhabitant-generator] you can
use the [createAndPopulateServiceLocator][createandpopulateservicelocator] method near the start of your main method, like this:
  
```java
  public static void main(String argv[]) {
      ServiceLocator locator = ServiceLocatorUtilities.createAndPopulateServiceLocator();
      
      MyService myService = locator.getService(MyService.class);
      
      ...
  }
```

[servicelocator]: apidocs/org/glassfish/hk2/api/ServiceLocator.html
[inhabitant-generator]: inhabitant-generator.html
[createandpopulateservicelocator]: apidocs/org/glassfish/hk2/utilities/ServiceLocatorUtilities.html#createAndPopulateServiceLocator()
[service]: apidocs/org/jvnet/hk2/annotations/Service.html
