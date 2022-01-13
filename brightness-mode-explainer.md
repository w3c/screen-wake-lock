# Design Document: the need to control screen brightness

_As discussed in the [DAS WG 2021-11-17](https://www.w3.org/2021/11/17-dap-minutes.html) meeting about Screen Brightness and the Screen Wake Lock API, the idea of this document is to make it easier for the meeting attendees to work on an explainer than if this had been committed to GitHub directly. Once this is done the goal is to move the contents to a proper explainer on GitHub._

Previously:
- https://github.com/WICG/proposals/issues/17
- https://github.com/w3c/screen-wake-lock/issues/129
- https://github.com/w3c/devicesensors-wg/issues/51
- https://www.w3.org/2021/11/17-dap-minutes.html

## Introduction

There is currently a need for web applications to increase the brightness of a device's screen.

This would help address the following use cases:
- a user scans QR/barcode under various lighting conditions.
- remote medical, where increasing the screen brightness could assist in remote examination
- biometric security apps, where increasing the screen brightness can help illuminate features to get better imagery from the front facing camera
- increasing contrast for readability for the visually impaired
- [make up mirror](https://play.google.com/store/apps/details?id=mmapps.mirror.pro&hl=en&gl=US) style apps

[Screen Wake Lock API issue 129](https://github.com/w3c/screen-wake-lock/issues/129) was filed back in 2018, and since then there has been feedback from developers working on multiple areas about the usefulness of an API that addresses this use case.

## Open design issues

- Things to consider in the spec proposal
  - UAs should be free to limit maximum brightness level and/or do other things like increase contrast.
  - Idea is to focus on mobile first (i.e. not worry about external displays). How to indicate that in the spec? Does it *need* to be indicated in the spec?
  - Should this trigger "`prefers-contrast: more`" in [CSS Media Queries](https://drafts.csswg.org/mediaqueries-5/#prefers-contrast)?
- From [https://github.com/w3c/devicesensors-wg/issues/51](https://github.com/w3c/devicesensors-wg/issues/51)
  - Brightness levels
    - How bright is too bright? Consider 100% brightness with HDR displays, for example.
    - Take a granular value or an enum?
    - Related to whether script should be allowed to reduce brightness.
  - Permission model
    - Probably require user gesture (request but not consume it).
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
- Handle error cases, where such requests are denied or not possible.

## Non-goals

- Reading screen brightness level.
- Adjusting display brightness level to arbitrary values (absolute or relative).
  - [\<video\> integration to allow e.g. granular brightness control](https://github.com/w3c/screen-wake-lock/issues/129#issuecomment-926603108) is an instance of the above. This is most likely better handled elsewhere and should probably be a UA-specific control.

## Proposed Solutions

This was initially discussed in [Screen Wake Lock API issue 129](https://github.com/w3c/screen-wake-lock/issues/129) back in 2018 and also proposed as a [separate API in WICG](https://github.com/WICG/proposals/issues/17). This topic was also discussed at multiple TPACs under the Devices and Sensors WG. As such, several different API shapes have been considered over the years.

At this point, none have been fully discarded, and we are open to feedback about adopting some of the ideas described here.

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

This was considered for a long time until the [2021-11-17 DAS WG meeting to discuss the topic](https://www.w3.org/events/meetings/0f623aa1-2026-4366-846b-c2faedda4180). The idea was to extend the existing Screen Wake Lock API and tie the change in brightness to a WakeLockSentinel. Something like:

``` javascript
const lock = await navigator.wakeLock.request({ increaseBrightness: true });
```

### navigator.screenBrightness

Originally proposed in [WICG issue 17](https://github.com/WICG/proposals/issues/17). The API proposed there had a larger surface that allowed reading the brightness value (leading to potential fingerprinting issues).

### requestFullscreen() integration

A mixture of [whatwg/fullscreen](https://fullscreen.spec.whatwg.org/) issues [185](https://github.com/whatwg/fullscreen/issues/185) and [198](https://github.com/whatwg/fullscreen/issues/198). The idea would be to do something like

``` javascript
body.requestFullscreen({ increaseBrightness: true });
```

To take advantage of existing UA privacy mitigations and UX indications that would show to the user that a web application is increasing the brightness, and leaving full screen mode would make it clear that the UA should stop increasing the device's screen's brightness level.

### getUserMedia() integration

Add something to getUserMedia() to bundle the request for brightness into the media capture request itself.

The idea was dropped because this feature is not doing media capture and the need to ask for camera permission before e.g. entering full screen mode [complicates some use cases](https://github.com/w3c/screen-wake-lock/issues/129#issuecomment-858790397).

### CSS property

Some form of "scannable element" property. When an element with said property is visible, the UA would take care of increasing the brightness level.

*Note: this is the least hashed out proposal at this point. It is unclear how this would work with zoom, permissions, low-battery mode, what happens when an element scrolls in/out of view, or even how to mitigate something like*

``` html
<body style="brightness: max">*
```
