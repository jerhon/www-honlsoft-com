+++ 
date = "2019-05-13"
title = "Setting Receive Timeouts on a Web Service Client in Spring Framework"
tags = ["java", "spring-framework", "soap"]
categories = ["blog"]
+++

I'm working on a project utilizing Java and Spring Framework. Recently, I had a situation where I was calling a SOAP web service was taking a very long time to complete.  This is a simple problem to resolve.  I just needed to increase the receive timeout.  However, the actual code to fix this was not quite as straight forward.

For my client code, I usually make a simple wrapper to make WebService calls by extending the WebServiceGatewaySupport class. From time to time, I consider using WebServiceTemplate as well, but extending WebServiceGatewaySupport provides me a bit more configurability.

In order to configure the timeout, I needed to provide a custom message sender and configure the timeout on it.  Since I was using HTTP as my transport, it meant using the SimpleClientHttpRequestFactory to do it.  Here's an example of the code. The main portion to consider is the call to "setReadTimeout".

```java
import java.io.ByteArrayOutputStream;
import java.io.IOException;

import org.springframework.http.client.SimpleClientHttpRequestFactory;
import org.springframework.oxm.jaxb.Jaxb2Marshaller;
import org.springframework.ws.client.WebServiceClientException;
import org.springframework.ws.client.WebServiceIOException;
import org.springframework.ws.client.core.support.WebServiceGatewaySupport;
import org.springframework.ws.client.support.interceptor.ClientInterceptor;
import org.springframework.ws.context.MessageContext;
import org.springframework.ws.transport.http.ClientHttpRequestMessageSender;

public class WebServiceClientImpl extends WebServiceGatewaySupport {

    public WebServiceClientImpl(String baseUri, Jaxb2Marshaller marshall, int readTimeout) {
        super();

        setDefaultUri(baseUri);
        setMarshaller(marshall);
        setUnmarshaller(marshall);
        
        SimpleClientHttpRequestFactory requestFactory = new SimpleClientHttpRequestFactory();
        requestFactory.setReadTimeout(readTimeout);
        setMessageSender(new ClientHttpRequestMessageSender(requestFactory));

        setInterceptors(new ClientInterceptor[] { new ClientInterceptor() {
            @Override
            public boolean handleRequest(MessageContext messageContext) throws WebServiceClientException {
                ByteArrayOutputStream os = new ByteArrayOutputStream();
                try {
                    messageContext.getRequest().writeTo(os);
                } catch (IOException e) {
                    throw new WebServiceIOException(e.getMessage(), e);
                }

                String request = new String(os.toByteArray());
                
                logger.trace("Request Envelope (" + baseUri + "): " + request);
                return true;
            }

            @Override
            public boolean handleResponse(MessageContext messageContext) throws WebServiceClientException {
                ByteArrayOutputStream os = new ByteArrayOutputStream();
                try {
                    messageContext.getResponse().writeTo(os);
                } catch (IOException e) {
                    throw new WebServiceIOException(e.getMessage(), e);
                }

                String response = new String(os.toByteArray());
                logger.trace("Response Envelope (" + baseUri + "): " + response);
                return true;
            }

            @Override
            public boolean handleFault(MessageContext messageContext) throws WebServiceClientException {
                return false;
            }

            @Override
            public void afterCompletion(MessageContext messageContext, Exception ex) throws WebServiceClientException {

            }
        }});
    }

    public Object callWebService(Object request){
        return getWebServiceTemplate().marshalSendAndReceive(request);
    }

}
```

While the code is just for the receive timeout, there are various other options that can be controlled via the [SimpleClientHttpRequestFactory](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/client/SimpleClientHttpRequestFactory.html). It's worthwhile to take a look through.

In retrospective, Spring Framework provides a cohesive set of functionality to speed up application development. However, the framework can get in its own way-- making simple tasks like this much more complex than needed.