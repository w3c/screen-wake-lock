#screen wake
An API to keep the screen from dimming / locking

##Use cases

* watch video without screen dimming (or worse, locking)
* user follows cooking recipe in Web browser, gets hands dirty, can't easily unlock screen when it locks
* a game use the device's accelerometer / gyroscope as a controler, without other user interaction; the screen shouldn't lock up
* a navigation Web app needs to continue to be visible for a while, even without direct user interaction
* a speech controled Web app in general would want to not be dependent on user touching the screen
* a score-keeper app for a table top game needs to stay awake to allow score visibility and infrequent score updates from players

## Existing similar APIs
See [list of mobile APIs for screen wake](https://github.com/w3c-webmob/web-api-gap/blob/master/features/screen-wake.md)

## Proposals
Proposals from the community.

### `navigator.poke`
A [proposed API](http://w3c.github.io/screen-wake/navigator_poke.html) that fulfills these use cases (as discussed in [specifiction](http://discourse.specifiction.org/t/allow-developers-to-control-wake-lock-aka-disable-auto-dimming).

### `screen.getWakeLock`
[API proposal](screen_requestWakeLock.md)

### `screen.requestWakeLock`
More similar to Firefox OS and Android [API proposal](screen_requestWakeLock_alternative.md)

### Declarative option
A member could be added to the app manifest to control this:

```JSON
{
  "bikeshed_me": "dont_sleep"
}
```

Or some other declarative means, like a HTML attribute `<html stayawake>`.




