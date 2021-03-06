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

# Events

The HK2 event service provides a pluggable mechanism that allows delivery of messages from service to
service.  Those services sending events are called publishers and those receiving events are called
subscribers.  A message can be any java object.

## Topics

A [Topic][topic] is a special HK2 service that can be injected into a service or can be looked up from
a [ServiceLocator][servicelocator].  It normally is injected as a ParameterizedType, where the generic
Type is the class of the event that will be published.  So if there is an event class called ImportantEvent
that is to be distributed to subscribers, the injection point for the [Topic][topic] might look like this:

```java
    @Inject
    Topic<ImportantEvent> importantEventTopic;
```

The publish method of [Topic][topic] will distribute the message to the set of subscribers.  In this way
the subscribers and the publishers need not have any previous knowledge of each other.

A [Topic][topic] may have qualifiers associated with it.  These qualifiers will normally be used to
further select the set of subscribers who should receive the message.  In the following example
only those subscribers who ask for CodeRed ImportEvent events will receive them (assuming CodeRed is
a qualifier):

```java
    @Inject @CodeRed
    Topic<ImportantEvent> importantEventTopic;
```

## Pluggability

In order to provide maximum flexibility in terms of qualities of service between publishers and
subscribers the HK2 event subsystem is pluggable.  This means that when a message is published
by the publish method of the [Topic][topic] service it is not given directly to the subscribers.
Instead there is an HK2 service called the [TopicDistributionService][topicdistributionservice]
that is responsible for distributing messages from the producer to the subscribers.

Having the [TopicDistributionService][topicdistributionservice] be responsible for the distribution
of messages allows the designers of the system to control the quality of service between the
producers and the subscribers.  Some of the qualities of service that can be controlled this way are
security, threading and error handling.  It is even possible to distribute events outside of the
JVM if that is what the [TopicDistributionService][topicdistributionservice] decides to do!

The [TopicDistributionService][topicdistributionservice] has a method called distributeMessage
which is responsible for sending events to subscribers.  It takes the [Topic][topic] that sent
the message and the message to send.  The [Topic][topic] contains information concerning the
Type of the [Topic][topic] and all of its associated qualifiers.

HK2 provides a default implementation of [TopicDistributionService][topicdistributionservice]
which will be described in the sections below.

## The Default TopicDistributionService

In order to enable the default [TopicDistributionService][topicdistributionservice] the
[ServiceLocatorUtilities][servicelocatorutilities] method enableTopicDistribution should be called:

```java
    public void initializeEvents(ServiceLocator locator) {
        ServiceLocatorUtilities.enableTopicDistribution(locator);
    }
```

The default [TopicDistributionService][topicdistributionservice] is named HK2TopicDistributionService
and has a priority of 0.  It is possible to enhance the behavior of the default
[TopicDistributionService][topicdistributionservice] by adding an implementation of
[TopicDistributionService][topicdistributionservice] with a higher priority and which injects
the default [TopicDistributionService][topicdistributionservice].  Then the default implementation
can be delegated to by the custom enhanced [TopicDistributionService][topicdistributionservice].
An example of how to do this is given [here][threaded-events-example].

## Subscribers

In the HK2 default system subscribers are found by finding methods on services that have a
single parameter annotated with [@SubscribeTo][subscribeto].  Any other parameters of the
method are considered to be injection points that should be satisfied by normal HK2 services.
The following methods are all subscribers to ImportantEvent:

```java
  public void importantEventSubscriber(@SubscribeTo ImportantEvent event) {
    // Do something with this important event
  }
  
  public void importantEventSubscriber(@SubscribeTo ImportantEvent event, ServiceLocator locator) {
    // Do something with this important event and with the ServiceLocator
  }
  
  public void importantEventSubscriber(ServiceLocator locator, @SubscribeTo ImportantEvent event) {
    // Do something with this important event and with the ServiceLocator
  }
```

All three of the above methods will be called when a [Topic][topic] publish method is called with
an ImportantEvent.

In the default implementation the subscribers will be called on the same thread as the caller of
the [Topic][topic] publish method.  Any exceptions thrown by subscribers (or for other reasons such
as an injection point not being available) will be given to all registered implementations of the 
[DefaultTopicDistributionErrorService][defaulttopicdistributionerrorservice].  The default
[TopicDistributionService][topicdistributionservice] will only distribute events
to services whose instances were created after the [ServiceLocatorUtilities][servicelocatorutilities]
method enableTopicDistribution has been called.  Any subscribers on a service that is disposed or
has had its associated descriptor removed will not be invoked.

Events will only be given to services that have already been created.  For example an event will
not cause a Singleton service that has not already been instantiated to become instantiated, even if
the Singleton service is a listener to the event.

A method parameter with [@SubscribeTo][subscribeto] can also take qualifiers.  A qualifier will
restrict the set of messages that will be given to the subscription method.  For example the
following method will only get ImportantEvents from Topics that are also qualified with
the CodeRed qualifier:

```java
  public void codeRedSubscriber(@SubscribeTo @CodeRed ImportantEvent redEvent) {
    // do something with the code-red event
  }
```

If an injection point on a subscription method has scope PerLookup then the instance of
that service will be disposed after the subscription method has returned.

## Conclusion

Events can help your application have low cohesion, as subscribers and publishers need not
have any relationship other than agreeing on an event Type.  HK2 provides a powerful mechanism
for plugging in any messaging provider to provide desired qualities of service.  A default
messaging provider is available which can either be replaced or enhanced if other qualities
of service are required by the application.

[topic]: apidocs/org/glassfish/hk2/api/messaging/Topic.html
[servicelocator]: apidocs/org/glassfish/hk2/api/ServiceLocator.html
[topicdistributionservice]: apidocs/org/glassfish/hk2/api/messaging/TopicDistributionService.html
[servicelocatorutilities]: apidocs/org/glassfish/hk2/utilities/ServiceLocatorUtilities.html
[subscribeto]: apidocs/org/glassfish/hk2/api/messaging/SubscribeTo.html
[defaulttopicdistributionerrorservice]: apidocs/org/glassfish/hk2/utilities/DefaultTopicDistributionErrorService.html
[threaded-events-example]: threaded-events-example.html
