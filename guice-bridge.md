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

# The Guice/HK2 Bridge

The Guice/HK2 Bridge can be used to inject [Guice][guice] services
into HK2 services or inject HK2 services into [Guice][guice] services.
It can be used bi-directionally as well.

+ [Definitions](guice-bridge.html#Definitions)
+ [Injecting Guice services into HK2 services](guice-bridge.html#Injecting_Guice_services_into_HK2_services)
+ [Injecting HK2 services into Guice services](guice-bridge.html#Injecting_HK2_services_into_Guice_services)
+ [Bi-Directional HK2 Guice Bridge](guice-bridge.html#Bi-Directional_HK2_Guice_Bridge)

## Definitions

+ A [Guice][guice] service is a service that is instantiated (created) by [Guice][guice]
+ An HK2 service is a service that is instantiated (created) by HK2

## Injecting Guice services into HK2 services

[Guice][guice] services can be injected into any injection point in HK2.
In order to do this you must tell HK2 about the [Guice][guice] [Injector][injector] 
which has the [Guice][guice] service definitions.
This is accomplished in two steps.

In the first step you should have the [ServiceLocator][servicelocator]
that contains services you wish to be injected with [Guice][guice] services.
You must initialize this [ServiceLocator][servicelocator] with some required
Guice/HK2 bridge services.  You can do this using the utility class [GuiceBridge][guicebridge].
This is a code snippet that initializes a [ServiceLocator][servicelocator]:
 
```java
  GuiceBridge.getGuiceBridge().initializeGuiceBridge(aServiceLocator);
```

In the second step you must tell your initialized [ServiceLocator][servicelocator] about the
specific [Guice][guice] [Injector][injector](s) that you want it to look for services in.
You do this with the [GuiceIntoHK2Bridge][guiceintohk2bridge] service that was added in the previous step.
The following code snippet adds a [Guice][guice] [Injector][injector] to be searched for services when injecting into HK2 services:
 
```java
  public void tieInjectorToLocator(ServiceLocator aServiceLocator, Injector guiceInjector) {
      GuiceIntoHK2Bridge guiceBridge = aServiceLocator.getService(GuiceIntoHK2Bridge.class);
      guiceBridge.bridgeGuiceInjector(guiceInjector);
  }
```

Any [Guice][guice] [Injector][injector] added with the [bridgeGuiceInjector][bridgeguiceinjector]
method will be searched for services that HK2 cannot otherwise find.

For example, if you have a service called GuiceService that is created by [Guice][guice], you can inject it into an HK2 service
(called HK2Service) like this:
 
```java
  @Service
  public class HK2Service {
      @Inject
      private GuiceService guiceService;
  }
```

## Injecting HK2 services into Guice services

HK2 services can be injected into [Guice][guice] services.
In order to do so, you must use the [HK2Inject][hk2inject] injection annotation.
For example, if you have an HK2 service named HK2Service that is to be injected
into a [Guice][guice] service (called GuiceService) your code would look like this:

```java
  public class GuiceService {
      @HK2Inject
      private HK2Service hk2Service;
  }
```

In order to do this we have provided an implementation of [Module][module]
that should be given to [Guice][guice] when creating the [Guice][guice] [Injector][injector].
This implementation of [Module][module] is [HK2IntoGuiceBridge][hk2intoguicebridge].
The following code snippet is an example of how you would create a [Guice][guice]
[Injector][injector] using the [HK2IntoGuiceBridge][hk2intoguicebridge] [Module][module] to
tell the [Guice][guice] [Injector][injector] about the [ServiceLocator][servicelocator] to use for finding HK2 services:
  
```java
  Injector injector = Guice.createInjector(
                new HK2IntoGuiceBridge(serviceLocator),
                // application modules);
```

Any [Guice][guice] service that can be created with this [Injector][injector] 
will now search the provided [ServiceLocator][servicelocator] 
when it encounters a service that is injected with the [HK2Inject][hk2inject] annotation.
  
## Bi-Directional HK2 Guice Bridge

[Guice][guice] and HK2 can bridge back and forth between each other.
The following code example shows how you could accomplish such a thing:

```java
  public Injector createBiDirectionalGuiceBridge(ServiceLocator serviceLocator,
            Module... applicationModules) {
        Module allModules[] = new Module[applicationModules.length + 1];
        
        allModules[0] = new HK2IntoGuiceBridge(serviceLocator);
        for (int lcv = 0; lcv < applicationModules.length; lcv++) {
            allModules[lcv + 1] = applicationModules[lcv];
        }
        
        Injector injector = Guice.createInjector(allModules);
        
        GuiceBridge.getGuiceBridge().initializeGuiceBridge(serviceLocator);
        GuiceIntoHK2Bridge g2h = serviceLocator.getService(GuiceIntoHK2Bridge.class);
        g2h.bridgeGuiceInjector(injector);
        
        return injector;
    }
```

The above method will create a [Guice][guice] [Injector][injector] 
where services created by Guice can be injected with HK2 services (using the [HK2Inject][hk2inject] annotation).
Also services created by HK2 can be injected with [Guice][guice] services (using any supported HK2 injection annotation).

[guice]: https://github.com/google/guice
[injector]: https://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/Injector.html
[servicelocator]: apidocs/org/glassfish/hk2/api/ServiceLocator.html
[guicebridge]: apidocs/org/jvnet/hk2/guice/bridge/api/GuiceBridge.html
[guiceintohk2bridge]: apidocs/org/jvnet/hk2/guice/bridge/api/GuiceIntoHK2Bridge.html
[bridgeguiceinjector]: apidocs/org/jvnet/hk2/guice/bridge/api/GuiceIntoHK2Bridge.html#bridgeGuiceInjector
[hk2inject]: apidocs/org/jvnet/hk2/guice/bridge/api/HK2Inject.html
[module]: https://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/Module.html
