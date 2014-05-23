# `screen.getWakeLock()` API

## Abstract

This method is used to request a wake lock on a top-level browsing context. This prevents the device from disables the device from entering a power-saving state (e.g, truning off the screen).

# Use cases

 * Applications that don't rely on finger/click based interaction, such as games that use the acceleromenter or voice input. 
 * Applications where it's critical for a user to be able to see the informations on screen while they are performing a different task. E.g., anapplication for preparing meals, or for fixing appliances.

## Example

```JS
if(screen.getWakeLock && screen.wakeLockState !== "locked"){
   screen.getWakeLock().then(letsGetThisPartyStarted)
}

//sometime later
if(screen.wakeLockState !== "locked"){
   screen.releaseWakeLock();
}

```

# Extensions to the `Screen` interface

```JS
partial interface Screen {
  promise getWakeLock(optional ScreenLockOptions options);
  promise releaseWakeLock()
  readonly WakeLockState wakeLockState;
};

enum WakeLockState{ "locked", "unlocked", "unknown"};

dictionary ScreenLockOptions{
  allowDim = false; 
};
```


