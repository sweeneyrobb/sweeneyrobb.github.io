---
title: "Application Navigation with the New Angular 2 Router"
layout: post
description: 
categories: [blog]
tags: [angular2]
comments: true
modified: 2016-06-24T09:45:23-07:00
---
We have been building an application on Angular 2, since Beta 8. Managing the way 
our application handles navigation has been somewhat static. Mostly a navigation 
component with static `routeLinks` littered throughout. 

In addition, setting up the router config was using the magic of reflection to yank 
in the routes. Making it somewhat difficult to pull out the `@RouteConfig` data 
for use in our application. Sure, you can reflect the data off of the component. 
But, this seems, somehow inefficient and overly complex.

## Out with the old, In with the new ##

More we've recently made the upgrade from RC1 to RC3. In this upgrade, it's seems 
as if the [official Angular 2 docs have been updated](https://angular.io/docs/ts/latest/guide/router.html)
to reflect usage of the 'new' router. Note the big disclaimer at the top.

>> The Component Router is in alpha release. This is the recommended Angular 2 router 
>> and supersedes the earlier deprecated beta and v2 routers.

There is also a great article on [thoughtram.io](http://thoughtram.io), 
[Routing in Angular 2 Revisited](http://blog.thoughtram.io/angular/2016/06/14/routing-in-angular-2-revisited.html), 
written by [Pascal Precht](http://twitter.com/pascalprecht), which does a great job 
outlining the new features of the new router.

With any new toy we wanted to take it out for a spin and see if we can't more simply 
solve some of our navigation issues, with something more native, using the new router.

## Gettings things setup ##

One of the most interesting things you'll find with the new router is that you can 
use a plain `<any>[]`. TypeScript will cast it to the correct `RouterConfig` type. 
Which boils down to an interface array of type `Route`. All of fields are marked 
nullable, which makes this very simple to configure.

Because this is an interface we can provide members that are not part of the `Route` 
type and the router won't get upset. This is super cool because now we can tie 
information that is relevant to the route, directly to the route config itself.

{% gist d2154bbd34a416642767c00f085442a5 app-routes.ts %}

As you can see in my example, we've attached members `name, description, showInNav` which 
are not required by the router. In order to feed this into the router, you'll need to use 
the `providerRouter()` and send the result into the providers array on your bootstrapper.

{% gist d2154bbd34a416642767c00f085442a5 main.ts %}

Viola! That's pretty much all you'll need to get things going.

## Generating some navigation ##

Now that the router is all wired up you can go upon your normal business using `routerLink` 
very similarly to how you used to. But, since this information is stored in an `<any>[]` 
why not use it as a data source for your navigation.

{% gist d2154bbd34a416642767c00f085442a5 nav.component.ts %}

As you can see it's fairly straight-forward to use the same `RouterConfig` supplied to
the router for consumption elsewhere. Now you could have done this before, however,
it required you to reflect data out of the `@RouteConfig` metadata. The new router makes 
this very much less painful.

## Just for kicks ##

Just for kicks, I wanted to exercise again, how cool it is to have this data readily 
available. We wanted the application header to respond to route events. Allowing us to 
create a consistent, and generic application header.

{% gist d2154bbd34a416642767c00f085442a5 header.component.ts %}

What we've done here was have our `HeaderComponent` subscribe to the events being 
emitted by the router. There are several types of events, but, we only care about 
the `RoutesRecognized` events. Simply filter them out and subscribe. When the `RoutesRecognized` 
events fire off, the `HeaderComponent` will retrieve the `Route.path` and find a 
match in our `AppRoutes` constant, and assign the result to the `routeData` member.

## Conclusion ##

We are very much impressed with how easy and flexible the new Angular 2 Router is. 
It allows for a bunch of new scenarios natively, without having to resort to "workarounds" 
or "hacks" to get the desired behavior.

If you are interested, the entire example is available [here](http://plnkr.co/edit/lomzsQ?p=preview). 
This example is forked from [Pascal Precht](http://twitter.com/pascalprecht) example mentioned earlier.
