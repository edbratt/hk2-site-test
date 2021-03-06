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

HK2 is an implementation of JSR-330 in a JavaSE environment.


[JSR-330](http://jcp.org/aboutJava/communityprocess/final/jsr330/) defines services and injection points that can be dynamically discovered at runtime and which allow for Inversion of Control (IoC) and dependency injection (DI).


HK2 provides an API for control over its operation and has the ability to automatically load services into the container.


It is the foundation for the GlassFish V3 and V4 application servers as well as other products.


HK2 also has powerful features that can be used to perform tasks such as looking up services or customizing you injections, as well as several extensibility features allowing the users to connect with other container technologies


The following list gives an overview of some of the things that can be customized or extended with HK2:
- Custom lifecycles and scopes
- Events
- AOP and other proxies
- Custom injection resolution
- Assisted injection
- Just In Time injection resolution
- Custom validation and security
- Run Level Services


Getting started
----------------

Read the [introduction](introduction.html) and [get started](getting-started.html) with HK2.


API overview
------------

[Learn](api-overview.html) more about the HK2 API, or [browse](apidocs/index.html) the javadoc.


Features
--------

[Learn](extensibility.html) more about the features of HK2


Integration
-----------

HK2 is well integrated with [Eclipse GlassFish](https://glassfish.org), [Spring](http://www.springsource.org) and others !
