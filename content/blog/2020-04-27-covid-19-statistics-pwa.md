+++ 
date = "2020-04-27"
title = "COVID-19 Statistics Web Application - Progressive Web Application"
tags = ["angular", "covid-19-statistics-application", "pwa"]
categories = ["blog"]
+++

In this post, I'm going to be turning the COVID-19 statistics web application I've been building into a Progressive Web Application (PWA).  PWAs are a modern way to provide an app like experience with a web application without having to build a native application or using a wrapper such as cordova to put it in the store.

In short the main experience I'm looking for is the ability to install the web app as an application on a phone.  Like many of the posts in this series, Angular makes converting a web application to a PWA incredibly easy.  This will also work on the desktop for modern browsers and OSes that support PWAs.

![COVID-19 Install Button](/images/covid-19-install.jpg)

## Brief Overview of PWAs

In order to make a web application work as a normal application on a phone, it needs to be able to be accessible even when there is no network connection or the network connection is degraded.  This doesn't meant that it needs to show all it's data-- just that it can load.  A service worker is used to provide this capability.  It can be used to cache files used by the web application to run.  If the network is down or unreachable, it can rely on it's cache to open the app, and the app will still run.

This is also used to improve experiences when working in a normal web browser.  In today's world, web applications are littered with assets and JavaScript libraries that are very large.  While caching can help, loading time can be expensive.  By using a service worker to cache these files, it can improve performance even when the web site isn't running as a normal PWA.

Finally to recognize that some website is a PWA and to describe some of it's capabilities.  For example, is there a certain orientation the web app should be used in, or what's the name of the application installed on the phone?  This can be done through a web manifest.  The manifest is referenced in the header of the webpage by a link element.

# Setting up the Angular Project

Angular makes it incredibly easy to build a PWA. On an Angular 8+ application run the command:

```bash
ng add @angular/pwa
``` 

It will add all the necessary files and set up the project to run the service worker, add the manifest file, link it in the index.html, and add several generic icons.

This by itself is usually enough to get an application to work as a PWA, but there are usually some other things to configure after this.

## Set up the Icons

The setup with the PWA will include a number of files named something like 'icon-*.png' that can be used as icons at various resolutions.  Just replace these with the desired icon for your application.

Keep in mind here, is that there are a few icons that are required to be provided.  Those are the 512x512 and 192x192 icons.  Any usage of those icons will be scaled to the other sizes if required.  Angular will produce a manifest with a variety of different sizes, the other sizes can be removed.  If you wish you could provide an icon for all of them, but it's not entirely necessary.

## Review the Generated Manifest

The manifest.webmanifest file added to the project has several settings in it, such as the name  of the application, some theme colors, etc.  Review through those and make sure they match for the application.  This also contains the reference to the icons for the application.  Be sure to remove any that aren't needed, with the exception of the 192x192 and 512x512 icons as those are required.

Here is my example manifest for the application:
```json
{
  "name": "COVID-19 Statistics",
  "short_name": "COVID-19 Stats",
  "theme_color": "#1976d2",
  "background_color": "#fafafa",
  "display": "standalone",
  "scope": "./",
  "start_url": "./",
  "icons": [
    {
      "src": "assets/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "assets/icons/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "maskable any"
    }
  ]
}
```

## Improve Error Handling When the Network is Down

While caching deals with the static assets of the application, it does not deal with any of the API calls.  The application will still make API calls, but if the APIs encounter an error, the front end just keeps spinning with progress indicators.  I'm going to update this to add an alert that is triggered when an API fails, with a way to attempt a refresh of the application.

Rather than trying to catch all errors from all APIs individually, I'm going to create an HTTP interceptor to capture an error on any API call and display a modal over the page to indicate there is an error.

An HTTP interceptor can be used to alter or inspect all HTTP requests made by an angular application.  These are particularly useful in cases where an API token needs to be provided, or if you want to provide a login page if an API returns a 401 Unauthorized response, or in cases where you want to log all the HTTP requests out of an application.

```typescript
@Injectable()
export class ApiErrorInterceptor implements HttpInterceptor {
  
  constructor(private service: AppAlertService) {}

  intercept(request: HttpRequest<unknown>, next: HttpHandler): Observable<HttpEvent<unknown>> {
    return next.handle(request)
      .pipe(catchError((err) => {
        // PERFORM ERROR HANDLING HERE...
        return throwError(err);
      }));
  }
}
```

To get it to take effect on all HttpClient's, it can be registered via a provider in an Angular module:

```typescript
{ 
    provide: HTTP_INTERCEPTORS,
    useClass: ApiErrorInterceptor,
    multi: true
}
```

## Testing the PWA Locally

When testing the PWA locally, beware, the typical ```ng serve``` command used to debug a web application locally doesn't work.  Instead run ```ng build --prod``` and use an http server such as the NPM ```http-server``` package against the dist folder to get it to work.

## Downsides to the Approach

There are a few downsides to this approach.  It assumes the folks are on the cutting edge browser and OS.  This is now adopted in most major browsers, but it is fairly new in the realm of web development.  

Browser vendors drive when an application can be installed or not.  This presents itself as some weird interactions particularly in cases where an application may be uninstalled and then installed again.  It sometimes will take several refreshes of the browser, and the cache and or service worker may need to get cleared forcibly to get it to show immediately.

This also lacks the luster of having "an app" in a store, and some business users just want that traditional listing to show up they can install as a status symbol.

## Wrapping Up

This will be the last post in this series about my COVID-19 tracker and likely the last feature I'll be adding.  It's been a fun little project.  I hope it's been useful for other folks as well.
