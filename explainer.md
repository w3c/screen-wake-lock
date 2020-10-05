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

* A recipe website that keeps the screen on while you are cooking.
* A web page that presents a bar code, which has to be physically scanned by
  another person.
* A web page that provides monitoring or dashboard-style functionality where the
  screen must be on at all times.
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
A previous iteration of this spec included both Screen and System wake locks,
which prevented the CPU from entering a deep power state. The latter has been
moved to a separate spec, and is out of scope for this explainer. For more
context, see:
* [Anyone implementing "system"
  lock?](https://github.com/w3c/screen-wake-lock/issues/232)
* [Break specification into
  levels](https://github.com/w3c/screen-wake-lock/issues/253)
* [Convert to purely screen wake
  lock](https://github.com/w3c/screen-wake-lock/pull/255)

## API
Users can request a screen wake lock, and if the request succeeds it can be
either manually released later or released automatically by the platform due to
OS-specific policies that might be in place. The code looks like this:

```js
const lock = await navigator.wakeLock.request("screen");

lock.addEventListener("release", doSomething);

// Later, check and release the lock if necessary.
if (lock.released === false) {
  await lock.release();
}
```

### Requesting permission to use WakeLocks
To use the API, one must request access via
`navigator.wakeLock.request("screen")` (`"screen"` is optional). This
returns a promise which, if allowed by the user, resolves with a
`WakeLockSentinel`.

```js
let sentinel;
try {
  sentinel = await navigator.wakeLock.request("screen")
} catch (err) {
  // Access denied, or something went wrong.
}
```

### WakeLockSentinel
A `WakeLockSentinel` object provides a handle to the lock that has been
acquired.

`WakeLockSentinel` has two read-only attributes for introspection: `released`
indicates whether the lock has already been released, `type` returns the lock's
type (which is always _"screen"_).

The `WakeLockSentinel`'s `.release()` method is used to release the platform
wake lock. It asynchronously dispatches a _"release"_ event that one can listen
for in order to know that the lock is no longer being held.

The lock might also be released automatically by the platform without
`.release()` being called (for example, when the page is no longer visible). The
_"release_" event is still emitted the same way.

## Security considerations
* To avoid possible user fingerprinting issues, `WakeLock.prototype.request()`
  does not indicate to API users whether the actual call to obtain a lock from
  the operating system has succeeded or not. In other words, successful
  `WakeLock.prototype.request()` calls might be granted, but are not guaranteed
  to keep the screen awake. Ultimately, it is up to the browser and/or operating
  system if a lock is honored or not.

* Screen Wake Locks can only be acquired and held if the document is visible. If
  `WakeLock.request()` is called while the document is hidden, the request will
  be denied with a `NotAllowedError`. If the page is hidden after a lock has
  been acquired, it will be released automatically.

* Screen Wake Locks can cause various device components such as display or CPU
  to operate at higher power levels than they otherwise would. This can lead to
  undesirable effects such as faster than normal battery charge depletion. This
  is particularly relevant to mobile devices, which may not have a stationary
  power source readily available.

## Stakeholder Feedback / Opposition
* Chromium has been shipping this API by default to users since version 84.
* Mozilla has deemed this specification [worth
  prototyping](https://github.com/mozilla/standards-positions/pull/299).
* Apple is not a part of the Devices and Sensors WG, but voiced
  [concerns](https://lists.webkit.org/pipermail/webkit-dev/2020-February/031081.html)
  about the impact this API can have on battery life.
