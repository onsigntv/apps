# Transportation Object

OnSign TV players inject a Javascript object, named `transportation` into all running Apps, which is accessible via the Window object.

> Please, note that the `transportation` object is not available on all platforms, during app previewing or app thumbnailing. Therefore, all uses need to be protected with the standard `try {} catch {}`.

### Transportation Object Model:

```javascript
  {
  "last_update": 1541691472,
  "provider": "provider_id",
  "version": 1,
  "vehicle": {
    "id": "bus_322",
    "apc_count": 12,
    "operator_id": "Jones",
    "stop_requested": true,
    "heading": 45,
    "speed": 80,
    "coordinates": {
      "lat": 32.7157,
      "long": 117.1611
    }
  },
  "trip": {
    "id": "t3B8d-sl17-p173-r86D",
    "label": "Central Station",
    "route_id": 1234,
    "route_short_name": "12",
    "route_long_name": "Highland Avenue",
    "deviation": 65
  }
}
```

## Transportation Object API

The following methods are available on the `transportation` object:

  * [`transportation.getData(name)`](#getData)
  * [`transportation.addEventListener(event, func)`](#addEventListener)
  * [`transportation.removeEventListener(event)`](#removeEventListener)

**Note: this object can be expanded in the future to contain other types of information.**

***
#### <a name="getData"></a>transportation.getData(name)
> **Requires: Android Player 9.2.X**

Returns the correspondent value of transportation object model, specified by key `name`.

An empty query like `transportation.getData('')` returns the entire transportation object model.

Example:

```javascript
  // In the example below, it is important to wrap the code using a try/catch statement since the transportation object might not be available;
  try {
    // Get entire transportation object model representation from transportation object.
    var transportationObject = transportation.getData('');
    // Log entire transportation object:
    console.log(transportationObject);
    // Log the trip object:
    console.log(transportationObject.trip);
  } catch (ex) {
    console.error('Transportation object not available');
  }
```

Also, values can be accessed separately, for example:

* `vehicle.id`
* `vehicle.coordinates`
* `trip.route_id`
* `trip.route_long_name`

Example:

```javascript
  // In the example below, it is important to wrap the code using a try/catch statement since the transportation object might not be available;
  try {
    // Get an object representing the `vehicle` data from the transportation object.
    var vehicle = transportation.getData('vehicle.id');
    // Log the vehicle id:
    console.log(vehicle.id);
    // Get an string representing the `route_long_name` data from the transportation.trip object.
    var trip_route_long_name = transportation.getData('trip.route_long_name');
    // Log the trip route long name:
    console.log(trip_route_long_name);
  } catch (ex) {
    console.error('Transportation object not available');
  }
```

***
#### <a name="addEventListener"></a>transportation.addEventListener(event, func)
> **Requires: Android Player 9.2.X**

Adds an event listener to observe data changes for an specific value of transportation model. When data for type `event` changes, the correspondent function `func` will be called.

Example:

```javascript
  // In the example below, it is important to wrap the code using a try/catch statement since the transportation object might not be available;
  try {
   console.log('Add event listener for vehicle.coordinates:') 
   f = function(key,value){ console.log(key, value); }
   transportation.addEventListener('vehicle.coordinates', f);
  } catch (ex) {
    console.error('Transportation object not available');
  }
```

***
#### <a name="removeEventListener"></a>transportation.removeEventListener(event)
> **Requires: Android Player 9.2.X**

Removes an associated event listener of type `event` that was previously added.

Example:

```javascript
  // In the example below, it is important to wrap the code using a try/catch statement since the transportation object might not be available;
  try {
    console.log('Remove event listener for vehicle.coordinates');
  transportation.removeEventListener('vehicle.coordinates'));
  } catch (ex) {
    console.error('Transportation object not available');
  }
```
