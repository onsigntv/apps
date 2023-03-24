# App Attributes API

<a name="attr-directive">Any app can manipulate Player Attributes through the [`signage object`](JSBRIDGE.md) methods [`signage.getPlayerAttribute("name")`](JSBRIDGE.md#getplayerattribute) and [`signage.setPlayerAttribute("name", "value")`](JSBRIDGE.md#setplayerattribute). However, by calling these methods, there's no guarantee that the Attribute you're trying to access actually exists.

If an app wishes to depend on a specific Player Attribute, you must declare an App Attribute in its template through the `{{ __attr__ }}` directive. This will require the end-user to connect the App Attribute to an existing Player Attribute of the same `type` when editing the app.

## `{{ __attr__ }}`

This directive works like a function, expecting the following parameters.

Attribute | Action
--------- | ---------
 `name`   | Name used to address the App Attribute in this app.
 `type`   | Represents what kind of value the attribute will have. See [`type`](#attribute-type) section for available attribute types.
 `mode`   | Attribute value access mode: `"r"` for read only, `"w"` for write only, `"rw"` for reading and writing to the attribute.
 `label`  | The suggested name for the attribute. Will be shown to the end-user when editing the app.
 `help`   | Optional text containing further information about the attribute. Will be shown to the end-user when editing the app.
 `optional` | By default the end-user will be required to connect all App Attributes to Player Attributes. Except when the `optional` attribute is present.

```jinja
  {{ __attr__(
      name="temperature",
      type="number",
      mode="rw",
      label="Player Temperature °C",
      help="Stores the current temperature on the player, in Celsius",
    )
  }}
  {{ __attr__(
      name="weather_cond",
      type="string",
      mode="rw",
      label="Player Weather Condition",
      help="Stores the current weather condition on the player",
      optional=True,
    )
  }}
```

### Attribute `type`

There are four possible types of value a Player Attribute can have:

Type             | Description
-----------------| -------------
 `"string"`      | A JavaScript [`string`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String).
 `"number"`      | A JavaScript [`number`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number).
 `"stringarray"` | An array containing only elements of type `string`.
 `"numberarray"` | An array containing only elements of type `number`.


## Accessing App Attributes

By including the [`{{ __loadsdk__ }}`](JSBRIDGE.md#loadsdk) directive on your template, attributes can be accessed and changed through the newly available   [`window.getAppAttribute("name", "defaultValue")`](#getappattribute) and [`window.setAppAttribute("name", "value")`](#setappattribute) methods.

These functions are available only after the [`"signageloaded"` event](JSBRIDGE.md#signageloaded-event) is fired.

**Important:** When accessing an attribute's value, there is no guarantee it will have a value for the current player. If the value is missing it is assumed to be `null`. You should also provide a default value for all attributes to make sure your app works as intended.


### <a name="isappattributeconnected">`window.isAppAttributeConnected("name")`

Returns a boolean indicating whether the given app attribute `"name"` was connected by the end user.


### <a name="getappattribute">`window.getAppAttribute("name", "defaultValue")`

Retrieves the value of the Player Attribute that is connected to the App Attribute called `"name"`. The type of the return value depends on the [`type`](#attr-directive) used when declaring this attribute.

Returns `"defaultValue"` when the attribute is not set for the current player or when the attribute was declared as [`optional`](#attr-directive) and was not connected by the end-user. If no `"defaultValue"` was provided, returns `null`. 

Requires attribute [`mode`](#attr-directive) to be `"r"` or `"rw"`.

If an App Attribute with the given `"name"` does not exist or its access [`mode`](#attr-directive) is set to `"w"` (write only), an error will be raised.

``` javascript
var currentTemperature = window.getAppAttribute("temperature", 25);
```

### <a name="setappattribute">`window.setAppAttribute("name", "value")`

Sets the value of the Player Attribute that is connected to the App Attribute called `"name"` to `"value"`.

Requires attribute [`mode`](#attr-directive) to be `"w"` or `"rw"`.

The value parameter must be a Javascript `number`, `string`, `Array<number>`, `Array<string>` or `null`, according to the type specified when declaring the app attribute with the [`{{ __attr__ }}` directive](#attr-directive). All attributes can be set to `null`, irrespective of their `type`.

If an App Attribute with the given name does not exist, the value type is incorrect or the attributes access [`mode`](#attr-directive) is set to `"r"` (read only), an error will be raised.

Keep in mind that, if the app attribute called `"name"` was declared as [`optional`](#attr-directive) and the end-user did not connect it to a Player Attribute, all attempts to set a value will be silently ignored.

```javascript
window.setAppAttribute("temperature", 30);
window.setAppAttribute("weather_cond", "cloudy");
```

### <a name="setappattributes">`window.setAppAttributes({"attr1": "value1", "attr2": "value2"})`

Helper to allow setting multiple attributes at once in an optimized way.

All the restrictions documented in [`window.setAppAttribute()`](#setappattribute) apply to each `[key, value]` pair in the given object.

```javascript
window.setAppAttributes({"temperature": 30, "weather_cond": "cloudy"});
```


### `appattrchanged` event

This event is fired when one of the App Attributes changes value.

There's two ways an App Attribute can change value: when the Player Attribute they are connected to is edited, or directly through the [`window.setAppAttribute("name", "value")`](#setappattribute) method.

```javascript
document.addEventListener("appattrchanged", function (event) {
  // Name of the attribute that changed value
  var attrName = event.detail.name;
  // New value of the attribute. Can be a string or a number.
  var attrValue = event.detail.value;
});
```
