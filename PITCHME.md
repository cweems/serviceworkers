# Service Workers for Public Service
<br>
@charlieweems

---
## What's a Service Worker?

* A Service worker is a javascript process that sits between the client and network.
* Service workers can intercept, cache, and respond with resources.
* Service workers enable better offline UX, also support push notifications and background sync.
---
## Why Service Workers for Civic Tech?

If you're building for underserved populations, you can't assume stable access to internet.
---
## How We Build at CfA

* Simple web apps: mostly HTML and CSS (as little JS as possible).
* Keep files as small as possible.
* Mobile-responsive design emphasis.
* Support for legacy browsers (IE9/10).
---
### We optimize for a low-quality but consistent connection on a legacy device.
That's not necessarily an accurate depiction of our users.
---
## A More Realistic Perspective

* Operating on a decent Android device with a modern browser.
* OK quality internet, but with inconsistent access.
* Load times and data use matter, but we're not rationing kilobytes.
---
# Examples Where a Service Worker Could Help
---
* A GetCalFresh applicant has a smartphone, but no data plan. They start their application on the wifi network at their public library, but need to leave before they can finish.

* A service worker + a single page app could display the additional questions, store responses, and sync data when a connection becomes available.
---
* A ClientComm Parole Officer conducts house visits in areas that have unreliable cell service. They still need to send a reminder about a court appointment to one of their clients.

* A service worker could store their message and post it when they return to an area with cell service.
---
* A Clear My Record assister works in an office where the internet frequently goes down for 1-2 hours while their overburdened IT team fixes things. The assister still needs access to their dashboard so that they can view basic information.

* A service worker could cache recently viewed pages and serve them back while the internet is offline.
---
## CanIUse Service Workers?
* Globally, 74% of browsers support service workers.
* Supported on Chrome, Firefox, and Opera.
* Not supported on IE, Edge, or Safari (but they're working on it)
---
## Disclaimers
* Requires SSL.
* You can't really count on storage size or storage being persisted.
* Things can get confusing really fast if you're not intentional about your caching strategy.
---
# [Demo](https://offline-news-service-worker.herokuapp.com/)
---
### Service Worker Lifecycle
![lifecycle](https://bitsofco.de/content/images/2016/07/Lifecycle-3.png)
<br>
Source: bitsofco.de
---
### Registering a Service Worker

```javascript
// application.js
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js', { scope: '/' }).then(function(reg) {

    if(reg.installing) {
      console.log('Service worker installing');
    } else if(reg.waiting) {
      console.log('Service worker installed');
    } else if(reg.active) {
      console.log('Service worker active');
    }

  }).catch(function(error) {
    console.log('Registration failed with ' + error);
  });
}```
@[2](Check that our browser supports service workers)
@[5-11](Console.log the service worker's registration state for demo purposes)
@[13-16](Logging out an error if the service worker failed to install)

---
### Registering Immediately

```javascript
// sw.js
self.addEventListener('install', event => {
  self.skipWaiting();

  event.waitUntil(
    // caching etc
  );
});
```
@[3](Tell the service worker to skip the waiting phase and activate immediately)
---

### Caching On Install
![Cache on Install](https://developers.google.com/web/fundamentals/instant-and-offline/offline-cookbook/images/cm-on-install-dep.png)
---
### Network falling back to cache
```javascript
// sw.js
self.addEventListener('install', function(event) {
  event.waitUntil(
    caches.open('mysite-static-v1').then(function(cache) {
      return cache.addAll([
        '/css/whatever-v3.css',
        '/css/imgs/sprites-v6.png',
        '/css/fonts/whatever-v8.woff',
        '/js/all-min-v4.js'
        // etc
      ]);
    })
  );
});
```
@[2](Listen for install lifecycle event)
@[3](Tell our event to wait until the cache is populated)
@[4](Open a cache called myste-static-v1)
@[5-11](Load static assets into the cache)

---
### Common Caching Stategies
---
### Cache falling back to network
![Falling back to network](https://developers.google.com/web/fundamentals/instant-and-offline/offline-cookbook/images/ss-falling-back-to-network.png)
---
### Cache falling back to network

```javascript
// sw.js
self.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.match(event.request).then(function(response) {
      return response || fetch(event.request);
    })
  );
});
```
@[2](On a fetch event)
@[4](Check our caches to see if they match the request)
@[5](Return from the cache if we have something, otherwise initiate the network request)
---
### Cache falling back to network
* What types of resources would you want to serve with this strategy?
---
### Network falling back to cache
![Falling back to cache](https://developers.google.com/web/fundamentals/instant-and-offline/offline-cookbook/images/ss-network-falling-back-to-cache.png)
---
### Network falling back to cache
```javascript
// sw.js
self.addEventListener('fetch', function(event) {
  event.respondWith(
    fetch(event.request).catch(function() {
      return caches.match(event.request);
    })
  );
});
```
@[2](Listen for network fetch events)
@[4](If the request fails, we catch)
@[5](Catch returns our caches that match the fetch event's requests)

---
### Network falling back to cache
* What's a good use-case for going to the network first?
* There's an issue with this approach
* It has to do with when our event respondWith throws a catch
---
### Better: Cache, then network
![Cache then network](https://developers.google.com/web/fundamentals/instant-and-offline/offline-cookbook/images/ss-cache-then-network.png)
---
### Cache, then network
```javascript
// application.js
var networkDataReceived = false;

loadingSpinner();

var networkUpdate = fetch('/data.json').then(function(response) {
  return response.json();
}).then(function(data) {
  networkDataReceived = true;
  updatePage();
});

caches.match('/data.json').then(function(response) {
  if (!response) throw Error("No data");
  return response.json();
}).then(function(data) {
  if (!networkDataReceived) {
    updatePage(data);
  }
}).catch(function() {
  return networkUpdate;
}).catch(showErrorMessage).then(stopSpinner);
```
@[2](Let's track whether or not we've received a network response)
@[4](Show a loading spinner/status to the user)
@[6-11](This is our network request)
@[7](If it's successful, it will return json)
@[8-11](On success, it updates our status variable, and then triggers something to update the page)
@[13-24](While our network request is running, get data from our cache)
@[14-15](Check to make sure we recieced a response from the cache)
@[17-19](Check that the network request hasn't already udpated our data)
@[20-21](We couldn't find an entry in our cache, so we'll just return netWorkUpdate)
@[22](If network update fails, we show an error message to the user)
---
```javascript
// sw.js
self.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.open('dynamic-content').then(function(cache) {
      return fetch(event.request).then(function(response) {
        cache.put(event.request, response.clone());
        return response;
      });
    })
  );
});
```
@[2-3](Again, listen for fetch events)
@[4](Cache for dynamic content)
@[5-8](Update cache with new content)
---
### [Background Sync Demo](https://jakearchibald-gcm.appspot.com/)
---
### Registering a Background Sync Event
```javascript
// application.js
navigator.serviceWorker.ready.then(function(registration) {
  registration.sync.register('send-message').then(function() {
    // registration succeeded
  }, function() {
    // registration failed
  });
});
```
@[3](Register the sync event on serviceworker creation)
---
### Handling the sync with a serviceworker
```javascript
// sw.js
self.addEventListener('sync', function(event) {
  if (event.tag == 'send-message') {
    event.waitUntil(sendEverythingInTheOutbox());
  }
});
```
@[2](Add serviceworker event listener)
@[3-5](Sync will retry with an exponential fallback until success)
---
### Questions
---
