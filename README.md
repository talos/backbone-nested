# Backbone-Nested

A plugin to make [Backbone.js](http://documentcloud.github.com/backbone) keep track of nested attributes.

## The Need

Suppose you have a Backbone Model with nested attributes.  Updating the nested attribute won't cause the model's `"change"` event to fire, which is confusing.

```javascript
var user = new Backbone.Model({
  name: {
    first: 'Aidan',
    last: 'Feldman'
  }
});

user.bind('change', function(){
  // this is never reached!
});

user.get('name').first = 'Bob';
user.save();
```

Wouldn't it be awesome if you could do this?

```javascript
user.bind('change:name.first', function(){ ... });
```

## Usage

1. Download the [latest version](https://github.com/afeld/backbone-nested/downloads), and add `backbone-nested.js` to your HTML `<head>`, **after** `backbone.js` is included.

    ```html
    <script type="text/javascript" src="backbone.js"></script>
    <script type="text/javascript" src="backbone-nested.js"></script>
    ```

2. Change your models to extend from `Backbone.NestedModel`, e.g.

    ```javascript
    var Person = Backbone.Model.extend({ ... });
    
    // becomes
    
    var Person = Backbone.NestedModel.extend({ ... });
    ```

3. Change your getters and setters to not access nested attributes directly, e.g.

    ```javascript
    user.get('name').first = 'Bob';
    
    // becomes
    
    user.set({'name.first': 'Bob'});
    ```

Best of all, `Backbone.NestedModel` is designed to be a drop-in replacement of `Backbone.Model`, so the switch can be made gradually.  See [here](https://github.com/afeld/backbone-nested/issues?labels=Parity) for any known feature-parity issues.

## Nested Attributes

`get()` and `set()` will work as before, but nested attributes should be accessed using the Backbone-Nested string syntax:

### 1-1

```javascript
// dot syntax
user.set({
  'name.first': 'Bob',
  'name.middle.initial': 'H'
});
user.get('name.first') // returns 'Bob'
user.get('name.middle.initial') // returns 'H'

// object syntax
user.set({
  'name': {
    first: 'Barack',
    last: 'Obama'
  }
});
```

### 1-N

```javascript
// object syntax
user.set({
  'addresses': [
    {city: 'Brooklyn', state: 'NY'},
    {city: 'Oak Park', state: 'IL'}
  ]
});
user.get('addresses[0].state') // returns 'NY'

// square bracket syntax
user.set({
  'addresses[1].state': 'MI'
});
```

### Change Events

`"change"` events can be bound to nested attributes in the same way, and changing nested attributes will fire up the chain:

```javascript
// all of these will fire when 'name.middle.initial' is set or changed
user.bind('change', function(){ ... });
user.bind('change:name', function(){ ... });
user.bind('change:name.middle', function(){ ... });
user.bind('change:name.middle.initial', function(){ ... });

// all of these will fire when the first address is added or changed
user.bind('change', function(){ ... });
user.bind('change:addresses', function(){ ... });
user.bind('change:addresses[0]', function(){ ... });
user.bind('change:addresses[0].city', function(){ ... });
```

## Special Methods

### remove()

Acts like `unset()`, but if the unset item is an element in an array, compact the array.  For example:

```javascript
user.get('addresses').length; //=> 2
user.remove('addresses[0]');
user.get('addresses').length; //=> 1
```

## Contributing

Pull requests are more than welcome - please add tests, which can be run by opening test/index.html.
