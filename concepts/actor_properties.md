## Actor Properties

Entities can keep track of persistent custom data called actor properties. The properties are initialized when the entity is created, and can be modified with [entity events](../events.md), and accessed in [molang](molang.md).

There are four different types of actor properties:
* `int`: a whole number integer value.
* `float`: a floating-point fractional number.
* `bool`: a simple boolean, either `true` or `false`.
* `enum`: one value from a custom fixed set of options.

Actor properties have similarities to [scoreboard values](scoreboard.md), [molang variables](molang.md), and dummy components like [`variant`](../components/variant.md) and [`is_baby`](../components/is_baby.md).

* Compared to scoreboard values, actor properties don't have to be integers, and can be assigned a value equal to a molang expression. They can also be accessed in client-side molang, if desired. However, they can't be accessed in command selectors or [raw text](raw_text.md) messages, or displayed in the places scoreboard objectives can.

* Compared to molang variables, actor properties persist when the entity is unloaded.

* Compared to dummy components, actor properties have no side-effects, and don't require a unique component group for every possible value.

The value of a property can be accessed with the `query.actor_property` query. It takes one parameter, the identifier of the property, and returns the current value. `int` and `bool`-type properties are converted to floats as typical for molang.

`query.has_actor_property` is the same, and returns `1.0` if the entity has any property with that identifier, and `0.0` otherwise.

---

## JSON Structure

Actor properties go inside a `properties` object inside the `description` of an entity's behavior file. Each key is the namespaced identifier of the property, and the matching value is an object containing these fields:

|Type|Name|Description|
|-|-|-|
|<img src="../icons/string.png" width=16>|`type`|Which kind of value this property can have (`int`, `float`, `bool`, or `enum`).|
|<img src="../icons/list.png" width=16>|`range`|For `int` and `float`-type properties. Two entries: the minimum possible value for the property, and the maximum. Any time the property would be set to a value outside of this range, it gets clamped to fit.|
|<img src="../icons/list.png" width=16>|`values`|For `enum`-type properties. List of strings, where each string is a possible value for this property.|
|<img src="../icons/int.png" width=16>/<img src="../icons/float.png" width=16>/<img src="../icons/bool.png" width=16>/<img src="../icons/string.png" width=16>|`default`|Default value this property will have when the entity is created. Can either be a literal value, or a [molang](molang.md) expression.|
|<img src="../icons/bool.png" width=16>|`client_sync`|Optional, defaults to false. When true, this property will be accessible with `query.actor_property` client-side as well as server-side. When false, it's only available server-side, and the client will always return `0.0` and show a content log warning.|

The only way to change the value of an actor property after the entity is created is with [entity events](events.md). Use an object called `set_actor_property`. Each key is the identifier of a property, and the matching value is a [molang](molang.md) expression.

---

## Additional Information and Bugs

For `int`-type properties, the minimum lower value in `range` is `-2147483648`, and the maximum upper value is `2147483647`.

Remember that when setting `enum`-type properties, any string literal must be wrapped in single quotes to produce a valid molang string.

The following problems will produce a content log warning and fail to load the property:
* Not providing a namespace in the identifier
* Leaving out any required field
* Including more than 32 actor properties on a single entity
* Including less than 2 or more than 16 values in `values`
* Putting a value in `values` that's longer than 32 characters
* Putting values in `range` or `values` that don't match the type of the property
* Putting any more or less than two values in `range`
* Making the second value in `range` greater than the first one
* Setting `default` as a literal (non-molang) value that's outside `range`, or doesn't match the type of the property

---

## Examples
One of each type:
```jsonc
"minecraft:entity": {
   "description": {
      "identifier": "test:monster",
      "is_spawnable": true,
      "properties": {
         // could use this to display bones on the model
         "test:horns": {
            "type": "int",
            "range": [0, 8],
            "default": "math.random(6, 8)",
            "client_sync": true
         },
         // could use this to change behavior
         "test:stamina": {
            "type": "float",
            "range": [0, 100],
            "default": 33.33
         },
         // could use this for a custom death animation
         "test:dead": {
            "type": "bool",
            "default": false,
            "client_sync": true
         },
         // could use this to set overlay color
         "test:color": {
            "type": "enum",
            "values": ["red", "green", "purple"],
            "default": "'green'",
            "client_sync": true
         }
      }
   }
}
```

Increment or decrement a property:
```jsonc
"events": {
   "gain_horn": {
      "set_actor_property": {
         "test:horns": "q.actor_property('test:horns')+1"
      }
   },
   "lose_horn": {
      "set_actor_property": {
         "test:horns": "q.actor_property('test:horns')-1"
      }
   }
}
```

Set multiple properties:
```jsonc
"events": {
   "full_power": {
      "set_actor_property": {
         "test:horns": 8,
         "test:color": "'red'",
         "test:stamina": 100
      }
   }
}
```
