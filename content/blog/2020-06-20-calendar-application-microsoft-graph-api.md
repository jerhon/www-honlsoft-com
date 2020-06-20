+++
date = "2020-06-20"
title = "Calendar Application - Accessing the Microsoft Graph API"
tags = ["pi-calendar", "raspberry-pi", "web-development"]
categories = ["blog"]
+++

In my previous post, I described setting up an Angular application for authentication through Azure Active Directory using Microsoft's MSAL library.  In this post, I will describe connecting to a Microsoft Graph API utilizing the sign in from the MSAL library.

## Angular HttpClient

Angular has a common helper class utilized in order to make API calls to external services.  It is HttpClient.  This has a lot of convenience methods to call make HTTP requests and call restful APIs.

This is great, because typically an outside library is needed to perform API requests and abstract away unfriendly or incompatible browser implementations.  Angular provides that out of the box.

In order to use this class, import the HttpClientModule in the module definition in the Angular application.

```typescript
// ... the rest of your module configuration here...
imports: [
    // ... some modules here 
    HttpClientModule
    // ... some more modules here
]
```

To use the HttpClient in code, it can be injected via a constructor, and called in a class.

A good pattern is to put all HTTP access code in it's own Angular service.

This is a bit of a more advanced example, but here I am getting the profile photo for my Microsoft profile.  I've simplified the call just for example here.  I'll write another post on how to take the image and display it in an image element.  The goal here is just to get the scaffolding together to make a successful API request.

```typescript
@Injectable({
  providedIn: 'root'
})
export class CalendarService {

  constructor(private client: HttpClient, private sanitizer: DomSanitizer) { 
  }

  getPhoto() {
    return this.client.get('https://graph.microsoft.com/beta/me/photo/$value', {responseType: "blob"});
  }
}
```

The HttpClient returns an Observable which can be subscribed to by whatever consumes the service and calls the method.  However, one of the key pieces still missing is authenticating to that Microsoft Graph API call.

## Providing Authentication Tokens in the HttpClient

The msal-angular library provides the MsalInterceptor.  This needs to included as a provider in your angular module definition.

```typescript
providers: [
    // ... more providers here ...
    {
      provide: HTTP_INTERCEPTORS,
      useClass: MsalInterceptor,
      multi: true
    },
    // ... more providers here ...
  ],
```

That's it!  With the configuration we've already put in place on the MsalModule, the HttpClient will now authenticate based on the configuration that had be set up in the previous post.

Under the covers this is using an HttpInterceptor.  An HttpInterceptor is a special interface used in Angular whereby an HTTP call made through an HttpClient, will be able to be inspected and potentially modified by the HttpInterceptor.  Since we've registered it in the providers, any HTTP request through an HttpClient should go through the MsalInterceptor to insert authentication information.

The HttpInterceptor is a really great construct.  It allows for things like centralized logging, error handling, authentication token injection, and so forth.

Just be careful, remember when scoping an HttpInterceptor to make sure if sending something such as authentication information or modifying an HTTP request, it is only done for those URLs to the resource you are requesting.  For example, it would be bad if an authentication token for the Microsoft Graph API were passed to a non-Microsoft endpoint.  It could have potentially shared an access token with a third party that could utilize it for nefarious purposes.

This is why in the previous article, there was the protected resources URLs.  These are there to limit exactly where the authentication token is sent.

```typescript
export const protectedResourceMap: [string, string[]][] = [
  ['https://graph.microsoft.com/v1.0/me', ['user.read', 'calendars.read', 'calendars.read.shared']],
  ['https://graph.microsoft.com/beta/me', ['user.read', 'calendars.read', 'calendars.read.shared']]
];
```

## Wrapping Up

In this article, I've described how to make an API call to the Microsoft Graph API and inserting authentication tokens in the request utilizing the msal-angular library.

In my next article, I'll get into how to use the blob from the API call and get out a URL that can be bound in an img.

If you are looking for more information on what other types of information you can get out of the Microsoft Graph API, here is a link to the documentation: [Microsoft Graph API Documentation](https://docs.microsoft.com/en-us/graph/api/overview?toc=.%2Fref%2Ftoc.json&view=graph-rest-1.0).

To see more posts about this project, checkout the [project page](/projects/pi-calendar).