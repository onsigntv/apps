# Signage Object

OnSign TV players inject a Javascript object, named `signage` into all running Apps, which is accessible via the Window object. Through this object, your apps will be able to retrieve useful information regarding the device, player and campaign being played at the moment.

> Please, note that the `signage` object is not available on all platforms, during app previewing or app thumbnailing. Therefore, all uses need to be protected with the standard `try {} catch {}`.

## Signage Object API

The following methods are available on the `signage` object:

  * [`signage.playbackInfo()`](#playbackInfo)
  * [`signage.width()`](#width)
  * [`signage.height()`](#height)
  * [`signage.isVisible()`](#isVisible)
  * [`signage.getCurrentPosition()`](#getCurrentPosition)
  * [`signage.getGeoLocation()`](#getGeoLocation)
  * [`signage.triggerInteractivity("value" [, {"param": "pvalue"}])`](#triggerInteractivity)
  * [`signage.stopCurrentCampaign()`](#stopCurrentCampaign)
  * [`signage.getPlayerAttribute("name")`](#getPlayerAttribute)
  * [`signage.setPlayerAttribute("name", "value")`](#setPlayerAttribute)

Additionally there are a few methods for Text-To-Speech, on players that support such functionality.

  * [`signage.ttsSetLanguage("en")`](#ttsSetLanguage)
  * [`signage.ttsSetPitch(0.9)`](#ttsSetPitch)
  * [`signage.ttsSetRate(1.5)`](#ttsSetRate)
  * [`signage.ttsSpeak("text to be spoken" [, {language: "es", rate: 0.8}])`](#ttsSpeak)
  * [`signage.ttsSilence(2000)`](#ttsSilence)
  * [`signage.ttsStop()`](#ttsStop)
  * [`signage.ttsFlush()`](#ttsFlush)

The following methods have been **deprecated** and will not be supported on future versions:

  * [`signage.playCampaign("campaignId")`](#playCampaign) *deprecated*

### <a name="playbackInfo"></a>`signage.playbackInfo()`

> **Requires: Android Player 5.3.5, Windows Player 5.0.0**

Returns a stringified JSON object that bundles information about the player and the current campaign. This object can be expanded in the future to contain other types of information.

Object Model:

```javascript
  {
    // Reason this campaign was played. If there is no reason, type is defined as "unknown"
    "reason": {
      "type": "time", // Other values: "touch", "key", "geo", "timeout", "ondemand", "api" and "unknown"
      "timestamp": 15467551, // unix timestamp
      // For type === "touch"
      "x": 230, // X,Y values returned are based on a virtual screen of 100000x100000 pixels.
      "y": 470,
      // For type === "key"
      "keys": "abcd",
      // For type === "api", when using /trigger/text or signage.triggerInteractivity("text")
      "value": "text",
      // For type === "geo"
      "lat": -27.5967811,
      "long": -48.5201524,
      "direction": "in", // other possible value is "out".
      // For type === "ondemand" or type === "api"
      "params": { // Always an object, contains extra parameters passed through the URL.
        "foo": "bar"
      }
    },
    "campaign": {
      "id": "13132",
      "name": "campaign name",
      "duration": 20, // in seconds
      "tags": ["tag1", "tag2"], // Always an array, even without tags,
      "attrs": { // Always an object, even without attributes.
        "key 1": "value 1"
      }
    },
    "player": {
      "id": "21313",
      "name": "player name",
      "version": "5.2.0-develop",
      "tags": ["tag1", "tag2"], // Always an array, even without tags.
      "attrs": { // Always an object, even without attributes.
        "key 1": "value1",
        "key 2": "value2"
      }
    }
  }
```

Example:

```javascript
  // In the example below, it is important to wrap the code using a try/catch statement for two reasons:
  // 1. The signage object might not be available;
  // 2. The JSON.parse() might throw an exception if data is not valid.
  try {
    // Get the stringified JSON object representing the `playbackInfo` data from the signage object.
    var data = signage.playbackInfo();
    // Parse data into a JSON object
    var playbackInfo = JSON.parse(data);
    // Log the player name:
    console.log(playbackInfo.player.name);
  } catch (ex) {
    console.error('Signage object not available or data var does not contain a valid JSON string.');
  }
```

### <a name="width"></a>`signage.width()`

Returns the region (where the App is being displayed) width in pixels.

Example:

```javascript
  console.log('Width:', signage.width());
```

> Please, note that the Webview viewport size might be different than the region resolution. "The screen density (the number of pixels per inch) on an Android-powered device affects the resolution and size at which a web page is displayed. The Android Browser and WebView compensate for variations in the screen density by scaling a web page so that all devices display the web page at the same perceivable size as a medium-density screen." More info [here](https://stuff.mit.edu/afs/sipb/project/android/docs/guide/webapps/targeting.html)

### <a name="height"></a>`signage.height()`

Returns the region (where the App is being displayed) height in pixels.

Example:

```javascript
  console.log('Height:', signage.height());
```

> Please, note that the WebView viewport size might be different than the region resolution. "The screen density (the number of pixels per inch) on an Android-powered device affects the resolution and size at which a web page is displayed. The Android Browser and WebView compensate for variations in the screen density by scaling a web page so that all devices display the web page at the same perceivable size as a medium-density screen." More info [here](https://stuff.mit.edu/afs/sipb/project/android/docs/guide/webapps/targeting.html)

### <a name="isVisible"></a>`signage.isVisible()`

> **Requires: Android Player 4.0.11, Windows Player 2.0.4**

Returns a boolean representing the app visibility.

Example:

```javascript
  console.log('App is visible?', signage.isVisible());
```

> OnSign TV players usually preload all campaign assets, including apps, a few seconds before starting the playback. As soon as the campaign starts the apps are already loaded, creating a better visual experience. More info at [Life cycle](#lifecycle)

### <a name="getCurrentPosition"></a>`signage.getCurrentPosition()`

> **Requires: Android Player 5.3.5**

Returns a stringified JSON object containing the player location data.
> Usually, it's better to use the native [HTML5 Geolocation API](https://developer.mozilla.org/en-US/docs/Web/API/Geolocation/Using_geolocation). However, under rare conditions, the native API may not return any data (e.g., when using an external GPS on Android.).

Position Model:

```javascript
  {
    coords: {
      accuracy: null,
      altitude: null,
      altitudeAccuracy: null,
      heading: null,
      latitude: 43.01256284360166,
      longitude: -89.44531987692744,
      speed: null
    },

    timestamp: 1479930166006
  }
```

Example:

```javascript
  try {
    // Get the stringified location data.
    var data = signage.getCurrentPosition();
    // Parse the location data into a Position object.
    // > The result object will be exactly the same as the one returned by the
    // native `Geolocation.getCurrentPosition()` method.
    var position = JSON.parse(data);
    // Log player's latitude and longitude
    console.log('Latitude:', position.coords.latitude);
    console.log('Longitude:', position.coords.longitude);
  } catch (ex) {
    console.log('Signage.getCurrentPosition() method is not available or data is not valid.');
  }
```

### <a name="getGeoLocation"></a>`signage.getGeoLocation()`

> **Requires: Android Player 9.9.5**

Returns a [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) that contains the player location, as good as can be determined.

In contrast to [`signage.getCurrentPosition()`](#getCurrentPosition), this method will fallback to the location configured in the Player Settings page or derived from the Player IP address. The position obtained by this method will be the same used to check for geographic restrictions on content publications.

```javascript
if (signage && signage.getGeoLocation) {
  signage.getGeoLocation().then(function(geo) {
    console.log(geo.src, geo.lat, geo.lng);
  });
}
```

When the promise is fulfilled the result will contain an object with three attributes: `lat`, `lng` and `src`.

```javascript
{
  "src": "ip", // can be one of "gps", "user", or "ip", based on the source of the location.
  "lat": -27.5967811, // player latitude
  "lng": -48.5201524, // player longitude
}
```

### <a name="triggerInteractivity"></a>`signage.triggerInteractivity("value" [, {"param": "pvalue"}])`

> **Requires: Android Player 9.8.6**

Triggers the Local API interactivity with user-defined "value".

For this method to have an effect a "Local API" interactivity needs to be defined – either on the player or the current campaign – with a regular expression that matches the parameter `"value"` of this method.

What will happen when this interactivity is triggered is defined in the Interactivity Configuration UI.

![Interactivity Sample](_screenshots/interactivity.png)

This method has an optional second parameter with a plain object containing keys and values that will be present in the [`signage.playbackInfo()`](#playbackInfo) of the triggered content, if any content is triggered.

For instance, in one app you can trigger an interactivity, passing an object containing parameters

```javascript
  try {
    signage.triggerInteractivity('show-content', {
      'content': 'contentValue'
    });
  } catch (ex) {
    console.error('Signage object not available');
  }
```

If the interactivity matches any configuration those parameters can be retrieved in the content that just started playing.

```javascript
  var playbackParams = {};

  // In the example below, it is important to wrap the code using a try/catch statement for two reasons:
  // 1. The signage object might not be available;
  // 2. The JSON.parse() might throw an exception if data is not valid.
  try {
    // Get the stringified JSON object representing the `playbackInfo` data from the signage object.
    var playbackInfo = JSON.parse(signage.playbackInfo());
    // Sanity checks to see whether the parameter was set.
    if (typeof playbackInfo.reason === 'object' && typeof playbackInfo.reason.params === 'object') {
      playbackParams = playbackInfo.reason.params;
    }
  } catch (ex) {
    console.error('Signage object not available or data var does not contain a valid JSON string.');
  }
  // If this content is playing because of that interactivity, this will be true.
  console.log(playbackParams['content'] === 'contentValue');
```

### <a name="stopCurrentCampaign"></a>`signage.stopCurrentCampaign()`

> **Requires: Android Player 8.1.3**

Stops the current campaign, moving to the next one in the loop. The campaign is reported as being partially played, so will only show in reports that have the "Include Partial Playback" option checked.


### <a name="getPlayerAttribute"></a>`signage.getPlayerAttribute("name")`

> **Requires: Windows Player 9.4**

Retrieves the current value of the player attribute called "name".

Attributes need to be created on the platform before they can be retrieved. If an attribute with that name doesn't exist or the value is not set for the current player this method will return `null`.

Otherwise it will return the value for the current player, which can be a Javascript `number` or `string`, according to the type specified when creating the attribute on the platform.


### <a name="setPlayerAttribute"></a>`signage.setPlayerAttribute("name", "value")`

> **Requires: Windows Player 9.4**

Sets the current value of the player attribute called `"name"` to `"value"`.

Player attributes need to be created on the platform before they can be set. The value parameter must be either a Javascript `number` or `string`, according to the type specified when creating the attribute on the platform.

If an attribute with the given name does not exist or the value type is incorrect, this call will have no effect.

Attributes set using this function are persisted only until the player reboots and affects attribute restrictions on content playback for this player until reboot.


## Text-To-Speech Engine

Text-To-Speech provides access to the underlying platform's ability to transform text into spoken word. Each text to be spoken is called an **utterance** and contains information about the language, pitch and rate of the text to be spoken.

All utterances are queued and spoken in [First-In-First-Out][1] order. To interrupt any utterance currently being spoken the `signage.ttsStop()` can be called. This will cause the TTS engine to start speaking the next queued utterance. If you want to make sure the engine is idle, call `signage.ttsFlush()` **before** calling `signage.ttsStop()`.


### <a name="ttsSetLanguage"></a>`signage.ttsSetLanguage("es-US-x-Voice1")`

> **Requires: Windows Player 10.1.0**

Sets the language, locale and voice that will be used for future [`ttsSpeak("text to be spoken")`](#ttsSpeak) calls that don't provide the `language` option.

The parameter is the [IETF language tag][2] followed by the voice name. Per the document, the two or three letter language code is to be picked from [ISO 639-1 or ISO 639-2][3]. The country code is picked from [ISO 3166-1][4]. The language tag and country subtag have to be separated using a hyphen.

The voice is platform dependent, goes from `Voice1` to `Voice9`, with `Voice1` being the default voice for the given locale. It is also used if the requested voice does not exist. To comply to the IETF format, the selected voice must be preceded by `-x-`, e.g. `"en-GB-x-Voice2"` or `"pt-BR-x-Voice3"`.

### <a name="ttsSetPitch"></a>`signage.ttsSetPitch(value)`

> **Requires: Windows Player 10.1.0**

Sets the pitch that will be used for future [`ttsSpeak("text to be spoken")`](#ttsSpeak) calls that don't provide the `pitch` option.

Value ranges from `0.0` to `4.0`. `1.0` is the normal pitch, lower values lower the tone of the synthesized voice, greater values increase it.

### <a name="ttsSetRate"></a>`signage.ttsSetRate(value)`

> **Requires: Windows Player 10.1.0**

Sets the rate that will be used for future [`ttsSpeak("text to be spoken")`](#ttsSpeak) calls that don't provide the `rate` option.

Value ranges from `0.0` to `4.0`. `1.0` is the normal speech rate, lower values slow down the speech (`0.5` is half the normal speech rate), greater values accelerate it (`2.0` is twice the normal speech rate).

### <a name="ttsSpeak"></a>`signage.ttsSpeak(text, [options])`

> **Requires: Windows Player 10.1.0**

If the Text-To-Speech engine is idle, begins speaking `text` immediately. Otherwise, enqueues `text` the utterance queue.

When provided, `options` is an object that specifies the behavior for this utterance. The available options are:

* `language`: overrides the language for the given text.
* `pitch`: overrides the pitch for the given text.
* `rate`: overrides the rate for the given text.


```javascript
signage.ttsSpeak("the train will arrive in 5 minutes", {language: "en-US"});
signage.ttsSpeak("el tren llegará en 5 minutos", {language: "es", rate: 1.1});
```

This function also supports the `signage.ttsSpeak(text, language)` signature for simplicity.

```javascript
signage.ttsSpeak("el tren llegará en 5 minutos", "es-ES");
```

Returns a [`Promise`][5] that will be *fulfilled* with a boolean parameter signifying whether the text was spoken until the end. The promise will be *rejected* when TTS engine is unable to speak the utterance due to an error.

```javascript
signage.ttsSpeak("speaking might fail").then(function(spoken) {
  if (spoken) {
    console.log("the phrase was spoken");
  } else {
    console.log("the phrase was interrupted or cancelled");
  }
}).catch(function() {
  console.log("failed to speak the phrase");
});
```

### <a name="ttsSilence"></a>`signage.ttsSilence(duration)`

> **Requires: Windows Player 10.1.0**

Causes the engine to pause for the given duration instead of speaking the next utterance. Duration is given in milliseconds.

Returns a [`Promise`][5] that will be *fulfilled* with a boolean parameter signifying whether the silence was maintained until the end. The promise will be *rejected* when TTS engine is unable to pause due to an error.

### <a name="ttsFlush"></a>`signage.ttsFlush()`

> **Requires: Windows Player 10.1.0**

Remove all enqueued utterances. If there are no utterance enqueued, calling this function is a no-op. The utterance currently being spoken, if any, will not be stopped. For that you must call `signage.ttsStop()`.

### <a name="ttsStop"></a>`signage.ttsStop()`

> **Requires: Windows Player 10.1.0**

Stops any utterance currently being spoken. If there are enqueued utterances, the oldest one is immediately dequeued and started. To completely stop the engine, call `signage.ttsFlush()` **before** calling `signage.ttsStop()`.

## Deprecated Methods

##### <a name="playCampaign"></a>signage.playCampaign("campaignId")

> **THIS METHOD IS DEPRECATED**. Please use [`signage.triggerInteractivity("value")`](#triggerInteractivity) instead.

> **Requires: Android Player 8.1.3**

Stops the current campaign, playing the campaign specified by the string `"campaignId"` instead. The currently playing campaign is reported as being partially played, so will only show in reports that have the "Include Partial Playback" option checked. After the campaign specified by `"campaignId"` plays, the next one in the loop will be played, as if interrupted campaign had reached its end.


[1]: https://en.wikipedia.org/wiki/FIFO_(computing_and_electronics)
[2]: https://en.wikipedia.org/wiki/IETF_language_tag
[3]: http://www.loc.gov/standards/iso639-2/php/English_list.php
[4]: http://www.iso.org/iso/english_country_names_and_code_elements
[5]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise
