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
