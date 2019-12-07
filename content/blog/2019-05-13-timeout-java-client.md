+++ 
date = "2019-05-13"
title = "Setting Receive Timeouts on a Web Service Client in Spring Framework"
tags = ["java", "spring-framework", "soap"]
categories = ["blog"]
+++

I'm working on a project utilizing Java and Spring Framework. Recently, I had a situation where I was calling a SOAP web service was taking a very long time to complete.  This is a simple problem to resolve.  I just needed to increase the receive timeout.  However, the actual code to fix this was not quite as straight forward.

For my client code, I usually make a simple wrapper to make WebService calls by extending the WebServiceGatewaySupport class. From time to time, I consider using WebServiceTemplate as well, but extending WebServiceGatewaySupport provides me a bit more configurability.

In order to configure the timeout, I needed to provide a custom message sender and configure the timeout on it.  Since I was using HTTP as my transport, it meant using the SimpleClientHttpRequestFactory to do it.  Here's an example of the code. The main portion to consider is lines 22-24.

<script src="https://gist.github.com/jerhon/d1be64a778666329922cae95f781815a.js"></script>

While the code is just for the receive timeout, there are various other options that can be controlled via the [SimpleClientHttpRequestFactory](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/client/SimpleClientHttpRequestFactory.html). It's worthwhile to take a look through.

In retrospective, Spring Framework provides a cohesive set of functionality to speed up application development. However, the framework can get in its own way-- making simple tasks like this more complex than needed.