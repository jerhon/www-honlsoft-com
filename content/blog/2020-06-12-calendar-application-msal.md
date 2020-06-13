+++
date = "2020-06-12"
title = "Calendar Application - Authenticating with MSAL"
tags = ["pi-calendar", "raspberry-pi", "web-development"]
categories = ["blog"]
image = "/images/msal-code.jpg"
+++

The first blog in this series is going to describe authenticating as a registered application in Azure Active Directory with Microsoft's MSAL package in Angular.  To see all blogs related to my Raspberry Pi calendar application, [check out the project page](https://www.honlsoft.com/projects/pi-calendar/).

There are a lot of great things about using an identity service:
* No on-premises passwords to secure!!
* Utilizes latest security standards
* Integration with other identity providers (Facebook, Google, Microsoft) logins without having to integrate with each service
* Great features such as two factor authentication, and others are implemented by the authentication provider (not me!)

The application will be integrating with a specific identity provider (Azure Active Directory).  Azure AD also provides other options such as B2C, and external identities which can also integrate with different providers.  I'm not going to go into all of those in this article, but just use a simple app registration.  I will not be describing how to set up an Application Registration in Azure.  [Microsoft and Azure do a great job of documenting this already.](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app)

## Add the MSAL Packages

Adding MSAL to an Angular application is very simple.  Starting with an Angular application, add the MSAL packages via command line.

```bash
npm install msal @azure/msal-angular --save
```

There are two libraries installed in the previous command.  msal is the base library that has a majority of the implement identity operations.  @azure/msal-angular is an wrapper with some helpful angular utilities around it.  Be careful though, there is an msal-angular package that's also hosted on NPM that is easy to confuse with the official @azure package.

There are three main pieces that are implemented in msal-angular I found useful for this project.

* MsalModule - The Angular module to import, it also can be used to supply the configuration.
* MsalGuard - A route guard that redirects to an Azure AD login for a route if the person is not authenticated.
* MsalInterceptor - Automatically injects authentication tokens on calls to the Microsoft Graph APIs.  I will be talking about that more in a future post.

## Setting up the MsalModule

Next, the MsalModule needs to be setup.  This includes some configuration about how to authenticate to the Azure App Registration.

```typescript
const isIE = window.navigator.userAgent.indexOf("MSIE ") > -1 || window.navigator.userAgent.indexOf("Trident/") > -1;

export const protectedResourceMap: [string, string[]][] = [
  ['https://graph.microsoft.com/v1.0/me', ['user.read', 'calendars.read', 'calendars.read.shared']],
  ['https://graph.microsoft.com/beta/me', ['user.read', 'calendars.read', 'calendars.read.shared']]
];

// ... in the module imports ...

MsalModule.forRoot(
  {
    auth: {
      clientId: environment.clientId,
      redirectUri: environment.redirectUri,
    },
    cache: {
      cacheLocation: "localStorage",
      storeAuthStateInCookie: isIE, // set to true for IE 11
    }
  },
  {
    popUp: isIE,   
    consentScopes: [
      "user.read",
      "openid",
      "profile",
      "calendars.read",
      "calendars.read.shared"
    ],
    unprotectedResources: ["https://www.microsoft.com/en-us/"],
    protectedResourceMap,
    extraQueryParameters: {}
  }
)
```

The main part here is the configuration.  Getting the client ID, redirect URL, and the right scopes.  The protected resource map will be used later with the MsalInterceptor to indicate which URLs need auth tokens.  The MsalInterceptor is used on the HttpClient to intercept any http calls and inject an auth token to the request.

There are two configuration objects. The tutorials online in the package didn't include the second  configuration. Without it, I had issues right away as the MsalGuard was looking for the configured scopes.  Once I added the parameters to the second configuration, it worked fine.

I have set up some of these configuration values in the environment.ts file, and made a separate environment for local development.  This way I can have two app registrations.  One for my hosted application, and one for local development.  This is to keep people from masquerading as my application and performing redirects on localhost for nefarious purposes.

The scopes define the permissions the application is allowed to use, and what is requested of a user. It will allow the application to read a user, the user's profile, and calendar.  When signing in for the first time the application will show a list of the of the permissions the application is requesting and ask for the user to approve.  To call additional Microsoft Graph APIs additional scopes would need to be added.

In the protected resources, you'll notice two URLs to the Microsoft Graph API.  The ```https://graph.microsoft.com/v1.0/me``` URL is their stable API.   The second ```https://graph.microsoft.com/beta/me```, contains beta features that have not yet made it to stable.  I had to use the beta APIs to return my personal profile picture.

## Using the MsalGuard to Protect Pages

This application uses Angular routing.  The first page contains the next event in the day, and the second page contains a list of all events for the day.  To protect those pages so that a user needs to be signed in, the MsalGuard can be used on the route guards.

Remember, this is a web application, EVERYTHING runs in the browser.  Even though a MsalGuard is in place doesn't mean the page is protected.  Someone could easily modify the code to circumvent that guard.  As always, protect any backend or APIs with the appropriate security.

```typescript
const routes: Routes = [
  { path: '', pathMatch:'full', component: DashboardComponent, canActivate: [MsalGuard] },
  { path: 'calendar', pathMatch:'full', component: CalendarComponent, canActivate: [MsalGuard]}
];
```

Note the ```[MsalGuard]``` included above in the canActivate configuration.  That is all it takes, now when a user hits the page, they will be forced to sign in if they are not already.  After signing in with a Microsoft / Active Directory login, the user will be prompted if they consent to the application.  The screen will look similar to what's below depending on the customizations behind the application.

![Permissions for the Angular Application](/images/calendar-application-permissions.jpg)

## Oustanding Issues

I have been having some intermittent issues regarding timeouts and retrieving tokens.  I thought it was part of the configuration I had set up, it seems a pretty common problem with the library right now.  [Here is the GitHub issue.](https://github.com/AzureAD/microsoft-authentication-library-for-js/issues/1222)

## Wrapping Up

This was the work it took to sign in via a Microsoft Identity to the application.  In the next article, I will be talking about setting up the MsalInterceptor to make API request to the Microsoft Graph APIs.
