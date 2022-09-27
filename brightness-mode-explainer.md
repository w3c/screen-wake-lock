# Design Document: the need to control screen brightness

## Introduction

Since 2018, the working group has received [significant requests](https://github.com/w3c/screen-wake-lock/issues/129) from web developers surrounding the need to increase the brightness of a device's screen.

For developers, it would help address the following use cases:

- a user scans a QR/barcode under various lighting conditions, in which case boosting the screen brightness creates more contrast.
  - this is already possible on native mobile apps such as [Starbucks'](https://play.google.com/store/apps/details?id=com.starbucks.mobilecard), [the UK Home Office's Document Check app](https://play.google.com/store/apps/details?id=uk.gov.HomeOffice.ho1) and the [Dutch government's CoronaCheck app](https://play.google.com/store/apps/details?id=nl.rijksoverheid.ctr.holder).
- increasing the screen brightness can help illuminate a user's facial features, improving the viability of certain application that rely on the front facing camera (e.g., [make up mirror](https://play.google.com/store/apps/details?id=mmapps.mirror.pro&hl=en&gl=US) style apps, and applications that can scan people's facial features to perform some particular action).

## Interested partners

- <https://iproov.com> authenticates users online for remote onboarding and verification, as well as kiosks inside banks and civic institutions. It also provides authentication for users in physical access settings.
- <https://xchart.com/> automates anesthesia charting. Increasing the screen brightness would allow customers to quickly enter "monitoring mode" where their laptop essentially becomes a patient vitals monitor display. In this mode, they'd like to present the option of increasing brightness. Additionally, there is a planned feature involving scanning barcodes on screens as a means of quickly opening a case on another device.

## Open design issues

The following issues remain open for discussion:
  - should UAs be free to limit the maximum brightness level and/or do other things like increase the contrast?
  - Should this be mobile first/only? (i.e., not worry about external displays)?
  - Should this trigger `prefers-contrast: more` in [CSS Media Queries](https://drafts.csswg.org/mediaqueries-5/#prefers-contrast)?
- From [https://github.com/w3c/devicesensors-wg/issues/51](https://github.com/w3c/devicesensors-wg/issues/51)
  - Brightness levels
    - How bright is too bright? Consider 100% brightness with HDR displays, for example.
    - Take a discrete or continuous value?
    - Related to whether script should be allowed to reduce brightness.
  - When dropping a screen brightness request
      - What about external displays? Do UAs need to keep track of levels for each display?

## Goals

- Provide the ability to request an increase to the display's brightness. This could be a UA-controlled maximum (which *could* be the maximum display brightness), either indefinitely or for a period of time.
- Provide the ability to release the request so that the device's brightness returns to its pre-request value (i.e., hand back control to OS).
- Handle error or low battery cases, where such requests are denied or not possible. For privacy, the API must not make it possible to determine whether a request failed due to low battery.

## Non-goals

- Reading the current screen brightness level.
- Adjusting the display brightness level to arbitrary values (absolute or relative).
  - [\<video\> integration to allow, e.g., granular brightness control](https://github.com/w3c/screen-wake-lock/issues/129#issuecomment-926603108) is an instance of the above. This is most likely better handled elsewhere and should probably be a UA-specific control.

## Proposed Solutions

The following represents some rough proposals that could address the use cases using various web technologies.  

We present them here only to foster discussion, and the working group has not settled on any particular one. We are open to feedback to pursue any of the proposals below. Or, if there is a better alternative solution we have not considered, we would be open to hearing it!

### Screen IDL interface extension

>🆕 This API shape is currently being experimented with.
>For WIP spec draft and Chromium CL, see https://github.com/beaufortfrancois/screen-brightness

Extend the [`Screen` interface](https://drafts.csswg.org/cssom-view/#the-screen-interface) with a new operation, inspired by the [Wake Lock API](https://developer.mozilla.org/en-US/docs/Web/API/WakeLockSentinel). Something like:

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

The idea was to extend the existing Screen Wake Lock API and tie the change in brightness to a `WakeLockSentinel`. Something like:

``` javascript
const lock = await navigator.wakeLock.request({ increaseBrightness: true });
```

This was considered for a long time until the [2021-11-17 DAS WG meeting to discuss the topic](https://www.w3.org/events/meetings/0f623aa1-2026-4366-846b-c2faedda4180).

### `navigator.screenBrightness`

The API proposed there had a larger surface that allowed reading the brightness value (leading to potential fingerprinting issues). Originally proposed in [WICG issue 17](https://github.com/WICG/proposals/issues/17).

### `requestFullscreen()` integration

The idea to integrate with `Element.requestFullscreen()`, to do something like:

```webidl
partial dictionary FullscreenOptions {
  boolean increaseBrightness = false;
};
```

Proposed usage:

```javascript
myButton.addEventListener("click", async () => {
  try {
    await document.body.requestFullscreen({ increaseBrightness: true });
    console.log("The display's brightness level has increased");
  } catch (e) {
    console.error(e);
  }
});
```

To take advantage of existing UA privacy mitigations and UX indications that would show to the user that a web application is increasing the brightness, and leaving full screen mode would make it clear that the UA should stop increasing the device's screen's brightness level.
See [whatwg/fullscreen](https://fullscreen.spec.whatwg.org/) issues [185](https://github.com/whatwg/fullscreen/issues/185) and [198](https://github.com/whatwg/fullscreen/issues/198).

### getUserMedia() integration

Add something to `getUserMedia()` to bundle the request for brightness into the media capture request itself.

This is complicated because this feature is not doing media capture (and there is no `MediaStream` to get out of screen brightness), and the need to ask for camera permission before, e.g., entering full screen mode [complicates some use cases](https://github.com/w3c/screen-wake-lock/issues/129#issuecomment-858790397).

### CSS property

Some form of "scannable element" property. When an element with said property is visible, the UA would take care of increasing the brightness level.

*Note: this is the least hashed out proposal at this point. It is unclear how this would work with zoom, permissions, low-battery mode, what happens when an element scrolls in/out of view, or even how to mitigate something like*

``` html
<body style="brightness: max">*
```

## Security considerations

- The API is available to secure browsing contexts.

- The screen brightness can be controlled only in response to a user gesture to ensure user retain control over their screen brightness. This prevents a situation where a site increases screen brightness every time the system or user overrides it manually.

- If the page visibility becomes hidden after screen brightness has been increased, the screen brightness should be restored automatically.

- To avoid possible user fingerprinting issues, when the request to control screen brightness is denied, a site should not be able to detect the exact reason why it happened. In other words, a generic `NotAllowedError` DOMException should be raised when there is no user gesture or battery level is too low for instance.

- If the screen brightness is at its maximum and a site requests a brighter screen, the request should succeed anyway so that it can't infer the screen brightness.

## Past discussions
- https://github.com/WICG/proposals/issues/17
- https://github.com/w3c/screen-wake-lock/issues/129
- https://github.com/w3c/devicesensors-wg/issues/51
- https://www.w3.org/2021/11/17-dap-minutes.html
- https://twitter.com/quicksave2k/status/1534867198342311936 (❤️ from web developers)

## Review requests
- [CSSWG](https://github.com/w3c/csswg-drafts/issues/6990)
  - [@tabatkins](https://github.com/w3c/csswg-drafts/issues/6990#issuecomment-1022678458) - in favour of a permissioned JS API over CSS property
- [Mozilla review](https://github.com/mozilla/standards-positions/issues/623) - in favour of similar solution to wake lock, i.e. Screen IDL interface extension with `requestBrightnessIncrease`
- [WebKit review](https://github.com/WebKit/standards-positions/issues/19)
- [Review feedback](https://github.com/w3c/screen-wake-lock/issues/335)
