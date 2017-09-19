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
  * [`signage.playCampaign("campaignId")`](#playCampaign)
  * [`signage.stopCurrentCampaign()`](#stopCurrentCampaign)

#### <a name="playbackInfo"></a>signage.playbackInfo()

> **Requires: Android Player 5.3.5, Windows Player 5.0.0**

Returns a stringified JSON object that bundles information about the player and the current campaign. This object can be expanded in the future to contain other types of information.

Object Model:

```javascript
  {
    // Reason this campaign was played. If there is no reason, type is defined as "unknown"
    "reason": {
      "type": "time", // Other values: "touch", "key", "geo", "timeout", "ondemand" and "unknown"
      "timestamp": 15467551, // unix timestamp
      // For type === "touch"
      "x": 230,
      "y": 470,
      // For type === "key"
      "keys": "abcd",
      // For type === "geo"
      "lat": -27.5967811,
      "long": -48.5201524,
      "direction": "in", // other possible value is "out".
      // For type === "ondemand"
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

#### <a name="width"></a>signage.width()

Returns the region (where the App is being displayed) width in pixels.

Example:

```javascript
  console.log('Width:', signage.width());
```

> Please, note that the Webview viewport size might be different than the region resolution. "The screen density (the number of pixels per inch) on an Android-powered device affects the resolution and size at which a web page is displayed. The Android Browser and WebView compensate for variations in the screen density by scaling a web page so that all devices display the web page at the same perceivable size as a medium-density screen." More info [here](https://stuff.mit.edu/afs/sipb/project/android/docs/guide/webapps/targeting.html)

#### <a name="height"></a>signage.height()

Returns the region (where the App is being displayed) height in pixels.

Example:

```javascript
  console.log('Height:', signage.height());
```

> Please, note that the WebView viewport size might be different than the region resolution. "The screen density (the number of pixels per inch) on an Android-powered device affects the resolution and size at which a web page is displayed. The Android Browser and WebView compensate for variations in the screen density by scaling a web page so that all devices display the web page at the same perceivable size as a medium-density screen." More info [here](https://stuff.mit.edu/afs/sipb/project/android/docs/guide/webapps/targeting.html)

#### <a name="isVisible"></a>signage.isVisible()

> **Requires: Android Player 4.0.11, Windows Player 2.0.4**

Returns a boolean representing the app visibility.

Example:

```javascript
  console.log('App is visible?', signage.isVisible());
```

> OnSign TV players usually preload all campaign assets, including apps, a few seconds before starting the playback. As soon as the campaign starts the apps are already loaded, creating a better visual experience. More info at [Life cycle](#lifecycle)

#### <a name="getCurrentPosition"></a>signage.getCurrentPosition()

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


#### <a name="playCampaign"></a>signage.playCampaign("campaignId")

> **Requires: Android Player 8.1.3**

Stops the current campaign, playing the campaign specified by the string `"campaignId"` instead. The currently playing campaign is reported as being partially played, so will only show in reports that have the "Include Partial Playback" option checked. After the campaign specified by `"campaignId"` plays, the next one in the loop will be played, as if interrupted campaign had reached its end.


#### <a name="stopCurrentCampaign"></a>signage.stopCurrentCampaign()

> **Requires: Android Player 8.1.3**

Stops the current campaign, moving to the next one in the loop. The campaign is reported as being partially played, so will only show in reports that have the "Include Partial Playback" option checked.
