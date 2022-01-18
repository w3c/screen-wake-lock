# Design Document: the need to control screen brightness

_As discussed in the [DAS WG 2021-11-17](https://www.w3.org/2021/11/17-dap-minutes.html) meeting about Screen Brightness and the Screen Wake Lock API, the idea of this document is to make it easier for the meeting attendees to work on an explainer than if this had been committed to GitHub directly. Once this is done the goal is to move the contents to a proper explainer on GitHub._

Previously:
- https://github.com/WICG/proposals/issues/17
- https://github.com/w3c/screen-wake-lock/issues/129
- https://github.com/w3c/devicesensors-wg/issues/51
- https://www.w3.org/2021/11/17-dap-minutes.html

## Introduction

Since 2018, the working group has received [significant requests](https://github.com/w3c/screen-wake-lock/issues/129) from web developers surrounding the need to increase the brightness of a device's screen.

For developers, it would help address the following use cases:

- a user scans QR/barcode under various lighting conditions, in which case boosting the screen brightness creates more contrast.
- remote medical, where increasing the screen brightness could assist in remote examination
- increasing the screen brightness can help illuminate a user's facial features, improving the viability of certain application that rely on the the front facing camera (e.g., [make up mirror](https://play.google.com/store/apps/details?id=mmapps.mirror.pro&hl=en&gl=US) style apps, and applications that can scan people's facial features to perform some particular action)
- increasing contrast for readability for the visually impaired


## Open design issues

The following issues remain open for discussion:
  - should UAs be free to limit maximum brightness level and/or do other things like increase contrast?
  - Idea is to focus on mobile first (i.e. not worry about external displays). How to indicate that in the spec? Does it *need* to be indicated in the spec?
  - Should this trigger "`prefers-contrast: more`" in [CSS Media Queries](https://drafts.csswg.org/mediaqueries-5/#prefers-contrast)?
- From [https://github.com/w3c/devicesensors-wg/issues/51](https://github.com/w3c/devicesensors-wg/issues/51)
  - Brightness levels
    - How bright is too bright? Consider 100% brightness with HDR displays, for example.
    - Take a discrete or continuous value?
    - Related to whether script should be allowed to reduce brightness.
  - Permission model
    - Would it require user gesture (request but not consume it)?
    - How to deal with multiple or external displays
  - While brightness changes
    - What if users change the brightness level while the lock is held?
  - When dropping screen brightness request
      - Does the UA have to restore the previous brightness level?
        - What about external displays? Do UAs need to keep track of levels for each display?
      - Should script be allowed to "hold the lock" indefinitely?

## Goals

- Provide the ability to request an increase to the display's brightness. This could be a UA-controlled maximum (which *could* be the maximum display brightness), either indefinitely or for a period of time.
- Provide the ability to release the request so that the device's brightness returns to its pre-request value (i.e. hand back control to OS).
- Handle error or low battery cases, where such requests are denied or not possible. For privacy, the API must not make it possible to determine whether a request failed due to low battery.

## Non-goals

- Reading screen brightness level.
- Adjusting display brightness level to arbitrary values (absolute or relative).
  - [\<video\> integration to allow e.g. granular brightness control](https://github.com/w3c/screen-wake-lock/issues/129#issuecomment-926603108) is an instance of the above. This is most likely better handled elsewhere and should probably be a UA-specific control.

## Proposed Solutions

The following represent some rough proposals that could address the use cases using various web technologies.  

We present them here only to foster discussion, and the working group has not settled on any particular one. We are open to feedback to pursuing any of the proposals below. Or, if there is a better alternative solution we have not considered, we would be open to hearing it!

### Screen IDL interface extension

Extend the [Screen interface](https://drafts.csswg.org/cssom-view/#the-screen-interface) with a new operation, inspired by the [Wake Lock API](https://developer.mozilla.org/en-US/docs/Web/API/WakeLockSentinel). Something like:

```webidl
[SecureContext]
partial interface Screen {
  Promise<ScreenBrightnessSentinel> requestBrightnessIncrease();
};

interface ScreenBrightnessSentinel {
  Promise<undefined> release();
};
```

Proposed usage:

```javascript
myButton.addEventListener("click", async () => {
  try {
    const sentinel = await screen.requestBrightnessIncrease();
    console.log("The display's brightness level has increased");
    // …and release it after 5s.
    window.setTimeout(() => {
      await sentinel.release();
    }, 5000);
  } catch (e) {
    console.error(e);
  }
});
```

### Screen Wake Lock integration

The idea was to extend the existing Screen Wake Lock API and tie the change in brightness to a WakeLockSentinel. Something like:

``` javascript
const lock = await navigator.wakeLock.request({ increaseBrightness: true });
```

This was considered for a long time until the [2021-11-17 DAS WG meeting to discuss the topic](https://www.w3.org/events/meetings/0f623aa1-2026-4366-846b-c2faedda4180).

### navigator.screenBrightness

The API proposed there had a larger surface that allowed reading the brightness value (leading to potential fingerprinting issues). Originally proposed in [WICG issue 17](https://github.com/WICG/proposals/issues/17).

### requestFullscreen() integration

The idea to integrate with `.requestFullscreen()`, to do something like:

``` javascript
body.requestFullscreen({ increaseBrightness: true });
```

To take advantage of existing UA privacy mitigations and UX indications that would show to the user that a web application is increasing the brightness, and leaving full screen mode would make it clear that the UA should stop increasing the device's screen's brightness level.
See [whatwg/fullscreen](https://fullscreen.spec.whatwg.org/) issues [185](https://github.com/whatwg/fullscreen/issues/185) and [198](https://github.com/whatwg/fullscreen/issues/198).
### getUserMedia() integration

Add something to getUserMedia() to bundle the request for brightness into the media capture request itself.

This is complicated because this feature is not doing media capture, and the need to ask for camera permission before e.g. entering full screen mode [complicates some use cases](https://github.com/w3c/screen-wake-lock/issues/129#issuecomment-858790397).

### CSS property

Some form of "scannable element" property. When an element with said property is visible, the UA would take care of increasing the brightness level.

*Note: this is the least hashed out proposal at this point. It is unclear how this would work with zoom, permissions, low-battery mode, what happens when an element scrolls in/out of view, or even how to mitigate something like*

``` html
<body style="brightness: max">*
```
