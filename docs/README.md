# Creating Your Own OnSign TV Apps

First thing one must know is that OnSign TV apps are nothing more than plain HTML5 pages. When the player is displaying an app it is actually showing a local webpage.

However, since static pages wouldn't suffice for most use cases, OnSign TV allows developers to [specify options](USERCONF.md#app-configuration) that can be chosen or filled in by the end-user, providing a safe way to act on their given values. There is also a [Javascript API](JSBRIDGE.md#signage-object) that can be used to gather information about playback and programmatically change your app based on how the campaign containing it was played.

## Table of Contents

  * [Introduction](#introduction)
  * [App Life Cycle](#app-life-cycle)
  * [Using Assets on Apps](#using-media-files-on-apps)
  * [Adding Configuration Options](USERCONF.md#app-configuration)
  * [Using the Javascript API](JSBRIDGE.md#signage-object)
  * [App Thumbnails](THUMBNAILING.md)
  * [App Simulator](https://github.com/onsigntv/app-simulator)

## Introduction

OnSign TV uses the Jinja templating language. It allows you to do things like iterating over values, calling functions and do all sorts of computation on user values. You can read more about it here: <http://jinja.pocoo.org/docs/dev/templates/>.

```html+jinja
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Custom App</title>
    <meta name="description" content="Long app description">
    <meta type="text" name="user_text" label="User text">
    <meta type="webfeed" name="feed" label="RSS feed URL">
  </head>
  <body>
    <ul id="feed">
    {% for item in feed %}
      <li>{{ item.title }}</li>
    {% endfor %}
    </ul>
    <p>Brought to you by:</p>
    <p>{{ user_text }}</p>
  </body>
</html>
```

Before rendering, the app is an *HTML5 template*, because it needs to be filled in with user values in order to be converted to HTML. Once the app is configured it becomes a regular HTML5 webpage.


### Identifying the App
There are two tags that are relevant to the app: the `<title>` and `<meta name="description">` tags. They allow you to change the app title and description that will be presented to the end-user. Please mind that according to the HTML5 standard the **title tag is mandatory** and OnSign TV will enforce that rule.

### Allowing Clicks or Touches

By default OnSign TV blocks any attempt from the user to interact with the app, either by clicking, touching or using a keyboard. If you wish to allow interaction with your app all you have to do is include the following tag in your HTML:

```html
<meta name="allow-interaction" content="yes">
```

## App Life Cycle

The following describes the life cycle of an application within the OnSign TV Player. Developers should have this in mind while developing an application.

`Preload` ➞ `Show` ➞ *`[Restart]`* ➞ `Destroy`

- `Preload`: The apps might be preloaded an unknown number of seconds before being displayed to the viewers. This preload time might vary from platform to platform. The standard load and DOMContentLoaded events can be used to keep track of this.
Make sure the first screen is ready before the show event, triggered by the next step in the application lifecycle. (i.e., The first article of a news app must be ready ASAP. The next article can be loaded in background while the first one is being displayed.)
- `Show`: The application is being displayed to the viewers. Applications that contain transitions, or timely events must start them at this point.
A news app that displays one article every X seconds, can only start it's internal timer once the show event is triggered within the HTML document. Also, since timers in Javascript can't be guaranteed, and there isn't a way to synchronize the Player and Javascript internal clocks precisely, waiting a second after the show event is received before initializing the timers is recommended.
- `Restart`: This is an optional event that is triggered within the HTML document if the campaign is being looped. It's up to the developer to make use of this or not, depending on the application.
- `Destroy`: The application is closed. No event is triggered at this point.

Example:

```html+jinja
<!DOCTYPE html>
<title>Sample App</title>

{# Insert a normalization layer into the App, which will make the custom (show/restart) #}
{# events to behave the same in all platforms, including when preview/thumbnailing the App #}
{{ __loadsdk__ }}

<script type="text/javascript">
  var log = document.getElementById('log');
  var start = Date.now();

  document.addEventListener('show', function() {
    var time = Date.now() - start;

    log.innerHTML += '<li>'+ time + 'ms: SHOW EVENT</li>';
  });

  document.addEventListener('restart', function() {
    var time = Date.now() - start;

    log.innerHTML += '<li>'+ time + 'ms: RESTART EVENT</li>';
  });

  document.addEventListener('measurechange', function(event) {
    var time = Date.now() - start;

    log.innerHTML += '<li>'+ time + 'ms: MEASURECHANGE EVENT '+ event.clientWidth +'x'+ event.clientHeight +'</li>';
  });

  window.addEventListener('DOMContentLoaded', function() {
    var time = Date.now() - start;

    log.innerHTML += '<li>'+ time + 'ms: DOMContentLoaded EVENT</li>';
  });

  window.addEventListener('load', function() {
    var time = Date.now() - start;

    log.innerHTML += '<li>'+ time + 'ms: LOAD EVENT</li>';
  });
</script>
```

## Using Media Files on Apps

Apps aren't only comprised of HTML5 files: you also need images, CSS and Javascript files. Since apps need to be shown even when there is no internet connection, you must upload all the assets needed to show your app to the same folder containing the HTML template of your app.

Once you upload them you must be able to reference them in your templates. All files you've uploaded are available for you under the `media` variable.

Say your HTML template *app.html* needs two files - *jQuery-2.0.1.js*, *Background_Image.png* - to work correctly and you already uploaded all three files to the same folder. Now you'd like to reference that script and that image on your app. Here is how you do it:

```html+jinja
<!DOCTYPE html>
<title>Example of referencing media</title>
<script type="text/javascript" src="{{ media.jquery_2_0_1.url }}"></script>
<style type="text/css" media="screen">
  body {
    background-image: url({{ media.background_image.url }});
    background-size: {{ media['Background_Image.png'].width + 20 }}px 100%;
  }
</style>
```

When using media please mind the following:

1. All files are accessed through the `media` variable. The name of the file is transformed from what you uploaded into a valid variable name. That means all text is lowercased, the extension is removed and all punctuation becomes underscores.
2. You can alternatively access files through their original name using the bracket syntax: `[]`. In our example both `media.background_image` and `media['Background_Image.png']` refer to the same file.
3. Files contain an `url` property, that will contain the *URL* to that file. This should the only way to reference a media inside your app and no manipulation of this value is allowed. Please mind that failing to use this variable will cause your app to be incorrectly rendered when sent to a user's player.
4. Image files will also contain two extra attributes: `width` and `height`, containing the respective width and height of the image, in pixels. You are allowed to do all sorts of mathematical manipulations with those values, useful when doing CSS adjustments.
