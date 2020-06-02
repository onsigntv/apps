## `Datasink` Overview

The `datasink` type is meant to allow a greater flexibility for the end-user. Instead of requiring the user to display data from Facebook, Instagram, Twitter and RSS in different apps, a **Data Sink** can be linked with any **Data Source** that provides the required fields.

**Data Sources** are a new feature of OnSign TV. This type is one of the most complex, because it allows the end user to provide data in a structured table format, either manually inputing data or automatically pulling it from Facebook, Instagram, Twitter, RSS and even another Data Source. Every entry in the Data Source can be edited and moderated by the end user, allowing total control of what is displayed in the app.

It is up to you, the app developer, to specify what kinds of data you are going to display. For instance, a "News App" might require a news title, news body, news image and publication date. But a "Menu Board App" it might only require a menu item name and price. Your task is to specify in the app what kind of data will be displayed and it's up to the user to match that requirement with a **Data Source** they have that data. If the users doesn't have one, it's easy to create a new **Data Source** with the data.


```html+jinja
<!DOCTYPE html>
<html>
  <head lang="en">
    <title>News App with Datasink</title>
    {{
      __datasink__(name='news', label='News Source', fields=[
        __field__(name='title', type='text', label='Headline Text', help='Keep text under 150 characters.'),
        __field__(name='image', type='image', label='Article Image'),
        __field__(name='published_at', type='date', label='Date of Publication', optional=true),
      ], help='Prefer choosing a news source with images for a better presentation.')
    }}
  </head>
  <script>
    function handleData(data) {
      for (var i = 0; i < data.source.length; i++) {
        console.log(data.source[i].title);
        console.log(data.source[i].image.url);
        console.log(data.source[i].published_at);
      }
      data.update.then(handleData);
    }
    window.addEventListener('DOMContentLoaded', function() {
      window.news.then(handleData);
    });
  </script>
</html>
```

This `datasink` will be displayed to the end-user as data source picker, like so:

![Example of data source picker](_screenshots/datasink.png)

Once connected to a data source, the data sink will be available in the browser [`window`][1] as a [`Promise`][2].

> **Heads up**: using a data sink automatically inserts a [Promise polyfill](https://github.com/taylorhakes/promise-polyfill) in the browser, in order to improve compatibility.


#### `__datasink__` Parameters

To declare a data sink you need to declare the following parameters. The `help` parameter is optional.

Parameter | Description
--------- | -----------
`name`    | Name of the data sink variable. Will be present in the browser [`window`][1] as a [`Promise`][2], resolved when the data is available.
`label`   | A label for the data sink that will be shown to the end-user to represent the data sink.
`fields`  | An array of `__field__` items, specifying all fields used by the data sink.
`help`    | Optional text containing further instructions on how to this data sink will be used, to aid the user in selecting a data source.


#### `__field__` Parameters

The number of `__field__` items for a given `__datasink__` is be based on the data required by the developer. Only `name`, `type` and `label` are required.

Parameter  | Description
---------- | -----------
`name`     | Variable name of the field, will be present in the resolved promise.
`type`     | Type of the field. Allowed types are  `"text"`, `"date"`, `"url"`, `"image"`, `"video"` or `"media"`. Type `"media"` means it can either be an `"image"` or a `"video"`.
`label`    | A label that will be shown to the end-user to aid in matching the requested field with a data source column.
`help`     | Optional text containing further instructions on how this value will be used, help the end-user.
`optional` | By marking a field as optional the end user link it with an optional column in the data source or not link it at all. When a field is not optional the end-user must link it to a required column and the app is guaranteed the field will be present in the resolved promise.


### Resolved Promise


Once the [Promise][2] is resolved, it will have the following data will be available on the callback:

```javascript
{
  "fields": {"field_name" : true, ... }, // fields that have been linked with a column will be true.
  "source": [ // every row in the data source will be an object with the field name and column values.
     {"field_name" : "field_data_1", ...},
     {"field_name" : "field_data_2", ...},
  ...],
  "update": new Promise(), // This Promise will resolve whenever there are updates to the data.
}
```

All fields that are marked as `optional` must be tested for presence before usage.

Fields of type `"text"` and `"url"` will have their value as a string.

Fields of type `"date"` fields will have their value as a string in the [ISO 8601 format][3]: `"2020-05-29T15:42:45Z"`.

Fields of type `image`, `video` or `media` it will have an object in the following format:

```javascript
  "field_name": {
      "type": "image" // or "video", used to disambiguate the type when needed.
      "url": "https://example.com", // URL to access the data.
      "width": 1920, // in pixels.
      "height": 1080, // in pixels.
      "duration": 11.2 // in decimal seconds, only available when "type" === "video".
  }
```


[1]: https://developer.mozilla.org/en-US/docs/Web/API/Window
[2]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise
[3]: https://en.wikipedia.org/wiki/ISO_8601
