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
## Registering a Service Worker

```javascript
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
@[1](Check that our browser supports service workers)
@[4-10](Console.log the service worker's registration state for demo purposes)
@[12-15](Logging out an error if the service worker failed to install)
