# `screen.requestWakeLock()` API

## Abstract

This method is used to request a wake lock on a top-level browsing context. This prevents the device from entering a power-saving state (e.g, truning off the screen).

# Use cases

 * Applications that don't rely on finger/click based interaction, such as games that use the acceleromenter or voice input. 
 * Applications where it's critical for a user to be able to see the informations on screen while they are performing a different task. E.g., an application for preparing meals, or for fixing appliances.

## Example

```JS
var lock;

if (screen.requestWakeLock) {
  screen.requestWakeLock().then(function(l) {
    lock = l;
  });
}

if (lock.isHeld()) {
    // do some work
}

// not necessary to release the lock,
// it will be release after desctruction
if (lock) {
  lock.release();
}
```

# Extensions to the `Screen` interface

```JS
partial interface Screen {
  // WakeLock object is returned in promise 
  promise requestWakeLock(optional ScreenLockOptions options);
};

dictionary ScreenLockOptions {
  allowDim = false;
};
```

# `WakeLock` object interface

```JS
interface WakeLock {
  void release();
  boolean isHeld();
};
```
