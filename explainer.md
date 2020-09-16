# Screen Wake Lock API

## Introduction
The **Screen Wake Lock API** allows web applications to prevent a device's
screen from dimming and locking.

See also:
* [The current specification](https://w3c.github.io/screen-wake-lock/)
* [web.dev article](https://web.dev/wakelock/) on the Screen Wake Lock API

## Why?
To avoid draining the battery, most devices quickly go to sleep when left idle.
While this is fine most of the time, some applications need to keep the screen
awake to complete their work. For example:

* A recipe app that keeps the screen on while you bake a cake or cook dinner.
* A boarding pass or ticket app that keeps the screen on until the barcode has
  been scanned.
* A kiosk-style app that keeps the screen on continuously.
* A web-based presentation app that keeps the screen on during a presentation.

Without this API, web developers have had to rely on libraries such as
[NoSleep.js](https://github.com/richtr/NoSleep.js), which essentially added a
[small
video](https://github.com/richtr/NoSleep.js/blob/eaf52afd1dfbb80145b4a39f3ec29307b80ab154/src/index.js#L37-L57)
to a page to keep the display on.

> Recent NoSleep.js releases have added support for the Screen Wake Lock API.

The purpose of this specification is to allow sites to be explicit about when
they want to acquire a wake lock so that browsers can provide better automated
and manual controls to prevent intentional and incidental issues.

## Non-goals
A previous iteration of this spec included both Screen and System wake locks.
The latter has been moved to a separate spec, and is out of scope for this
explainer.

## API
The API surface is not big: users can request a screen wake lock; if the
request succeeds it can be either manually released later or released
automatically by the platform due to OS-specific policies that might be in
place. The code looks like this:

```js
const lock = await navigator.wakeLock.request("screen");

lock.addEventListener("release", () => {
  console.log(`Has the lock been released? ${lock.released}`);
});
await lock.release();
```

### WakeLock
The `WakeLock` object is part of `Navigator`. It has a single method,
`request(type)`. The `type` argument is of the `WakeLockType` enum type, which
right now has a single value, _"screen"_.

`WakeLock.request()` performs permission checks, verifies that the document
issuing the request is visible and, if all checks succeed, asynchronously
requests a lock from the underlying platform to prevent the screen from turning
off.

A successful `WakeLock.request()` invication returns a promise that will
ultimately resolve with a `WakeLockSentinel` object.

### WakeLockSentinel
A `WakeLockSentinel` object provides a handle to the lock that has been
acquired.

`WakeLockSentinel` has two read-only attributes for introspection: `released`
indicates whether the lock has already been released, `type` returns the lock's
type (it is always _"screen"_ at the moment).

The `WakeLockSentinel.release()` method is used to release the platform wake
lock. It asynchronously emits a _"release"_ event that users can listen for in
order to know that the lock is no longer being held.

The lock might also be released automatically by the platform without
`WakeLockSentinel.release()` being called (for example, when the page is no
longer visible). The _"release_" event is still emitted the same way.

## Security considerations
* To avoid possible user fingerprinting issues, `WakeLock.request()` does not
  indicate to API users whether the actual call to obtain a lock from the
  operating system has succeeded or not. In other words, successful
  `WakeLock.request()` calls must be treated by users as **advisory-only**.

* Screen Wake Locks can only be acquired and held if the document is visible.
  If `WakeLock.request()` is called while the document is hidden, the request
  will be denied with a `NotAllowedError`. If the page is hidden after a lock
  has been acquired, it will be released automatically.

* Application of a wake lock causes various device components such as display
  or CPU to operate at higher power levels than they otherwise would. This can
  lead to undesirable and potentially dangerous effects such as excessive
  heating and faster than normal battery charge depletion. The latter is
  particularly relevant to mobile devices which may not have a stationary power
  source readily available.

## Stakeholder Feedback / Opposition
* Chromium has been shipping this API by default to users since version 84.
* Mozilla has deemed this specification [worth
  prototyping](https://github.com/mozilla/standards-positions/pull/299).
* Apple is not a part of the Devices and Sensors WG, but voiced
  [concerns](https://lists.webkit.org/pipermail/webkit-dev/2020-February/031081.html)
  about the impact this API can have on battery life.
