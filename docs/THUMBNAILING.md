# App Thumbnails

After an [app is configured](USERCONF.md) a thumbnail - or screenshot - of the chosen configuration is taken. That thumbnail is used to preview the app and allow the user to quickly identify which app instance to publish to their campaigns.

By default all thumbnails are taken 3 seconds after the `load` event. That means the end-user might have to wait at least 3 seconds to save the app and have a thumbnail. It also means that an animation might be happening and the app might not look its best, so here the techniques one can use to avoid that.

**Please mind there is a hard-deadline of 30 seconds for the thumbnail. After that time elapses it will fall back to the thumbnail of the app itself, which might not match the configuration a user has chosen.**

### Detect when a thumbnail is being taken

Because the same exact HTML previewed to the user is sent to the thumbnail service there is no way to detect that through Jinja (the template language). The only way to detect is through Javascript when the HTML is loaded.

You can check for the `navigator.webdriver` property. If it is truthy, then a thumbnail is being taken:

```html
<!DOCTYPE html>
<style>
  @keyframes blinker {
    50% { opacity: 0; }
  }
  .animate p {
    animation: blinker 1s linear infinite;
  }
</style>
<body class="animate">
  <p>Some blinking content</p>
  <script type="text/javascript">
    if (navigator.webdriver) {
      // The app is being thumbnailed. Stop all animations.
      document.body.classList.remove('animate');
    }
  </script>
</body>
```

### Specify when the thumbnail should be taken

By default there is a 3 second delay after `window.onload` is called, before the thumbnail is actually taken. That might be too much or too little, depending on how the app works.

To allow the app to dynamically inform when the thumbnail should be taken the app needs to inform two things:

1. That this app is capable of informing when it is ready. You can do that adding this meta element to it: `<meta name="caps" content="screenshot">`
1. That the app is ready to take the screenshot. You can do that setting the `ID` of any element to `signage-take-screenshot`.

```html
<!DOCTYPE html>
<meta name="caps" content="screenshot">
<body>
  <p>Some loaded content: <span class="name"></span></p>

  <script type="text/javascript">
    var request = new XMLHttpRequest();
    request.open('GET', 'http://example.com/sample.json', true);
    request.onload = function() {
      if (this.status >= 200 && this.status < 400) {
        var data = JSON.parse(this.response);
        // Fill in the loaded data.
        document.querySelector('.name').innerText = data.name;
        // Allow the browser to render the content before thumbnail.
        setTimeout(function() {
          document.body.id = 'signage-take-screenshot';
        }, 10);
      }
    };
    request.send();
  </script>
</body>
```

## Add a static thumbnail

Some apps might prefer to not have a dynamic thumbnail, opting for a static one. This can be done by having special files in the same folder as the HTML file when importing the app. There are two different options for static thumbnails:

1. If the static thumbnails is only to be used for the imported app itself, going through the regular thumbnail process when an end-user configures the app, then you should do this:

    When importing an app, upload a file called `TEMPLATE_NAME.thumb.png` to the same folder of your template. If your template is called `app.html`, then the file should be `app.thumb.png`.

2. If both the imported app and the user-configured app should use the same static thumbnail, what we call **sticky thumbnail**, then you should do this:

    When importing the app, upload a file called `TEMPLATE_NAME.sticky_thumb.png` to the same folder of your template. If your template is called `app.html`, then the file should be `app.sticky_thumb.png`.

Please mind that [audio apps](README.md#creating-an-audio-app), [automation apps](README.md#creating-an-automation-app) and [plugin apps](README.md#creating-a-plugin-app) require a sticky thumbnail to be imported.
