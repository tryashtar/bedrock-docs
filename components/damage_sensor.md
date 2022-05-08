## `minecraft:damage_sensor`

This component is used to detect, modify, or suppress damage dealt to an entity.

It consists of one or more damage triggers. When the entity is dealt damage, the first trigger that matches the damage will take effect. It could run an [entity event](../events.md), change the amount of damage taken, or even prevent it altogether.

After one trigger activates, any subsequent triggers are ignored, even if they would otherwise match the damage.

---

The component has only one field: <img src="../icons/object.png" width=16>/<img src="../icons/list.png" width=16> `triggers`. It can either be a single trigger object, or a list of them. A trigger object has several optional fields.

By default, a trigger will activate for *all* kinds of damage. There are two fields that allow for more specific control:

|Type|Name|Description|
|-|-|-|
|<img src="../icons/string.png" width=16>|`cause`|Only activate for [damage of a specific type](../damage-types.md).|
|<img src="../icons/object.png" width=16>/<img src="../icons/list.png" width=16>|`filters`|Only activate when the [filter](../filters.md) passes. This goes inside an `on_damage` object alongside `event`. The `has_damage` test is often used here to check for `fatal` damage.|

If the checks pass, the trigger will activate. There are three fields that can modify the damage in response:

|Type|Name|Description|
|-|-|-|
|<img src="../icons/bool.png" width=16>|`deals_damage`|Defaults to true. When set to false, the damage is completely canceled.|
|<img src="../icons/int.png" width=16>|`damage_modifier`|Added to the damage taken. Can be made negative to reduce taken damage, but can't reduce it below 0.|
|<img src="../icons/float.png" width=16>|`damage_multiplier`|Multiplies the damage taken, after `damage_modifier`. Can't reduce it below 1, unless `damage_modifier` already did.|

Lastly, there are two fields that can be used to perform some action in response to the damage:

|Type|Name|Description|
|-|-|-|
|<img src="../icons/object.png" width=16>|`event`|Run an [entity event](../events.md). This goes inside an `on_damage` object alongside `filters`.|
|<img src="../icons/string.png" width=16>|`on_damage_sound_event`|[Sound event](../sound-events) to play. This plays in addition to the entity's normal hurt sound. It can either be an entry in `individual_event_sounds`, or the entity's own `entity_sounds`.|

In `filters` and `event`, the `other` subject represents the attacker, if the damage was caused by an entity. The `damager` subject represents the entity that dealt the damage directly. For melee attacks, these are the same. For a ranged attack, for example, `other` would be the skeleton, while `damager` would be the arrow.

---

## Additional Information and Bugs
* The official docs say that `damage_modifier` is a float. This is effectively false; any value provided after a decimal point is ignored.

* Reducing damage with `damage_modifier` and `damage_multiplier` can cause problems with the damage invulnerability cooldown.
  * Whenever an entity *receives* damage and gets hurt (typically turning red and making a noise), the game saves the damage amount.
  * For the next second, if any incoming damage is *less* than this amount, it will be ignored. If any incoming damage is *greater* than the amount, the difference between the two will be taken instead.
  * When reducing damage with these fields, the incoming damage will always be greater than the received damage. Therefore, the entity will be eligible for getting hurt every tick, but will ultimately take no damage.
  * <p><video src="../videos/damage_modifier.mp4" width="400"/></p>

* If `damage_modifier` results in zero taken damage, the entity will still get hurt, except when it's `fall` damage. This is different from setting `deals_damage` to false.

* Sometimes it's desirable to run more than one [event](../events.md) in reponse to damage, such as running an event on both `self` and `attacker`. It's possible to use the `trigger` field in the referenced event to run a separate event with a different subject.

---

## Examples
Take double damage from tridents:
```json
"minecraft:damage_sensor": {
   "triggers": {
      "on_damage": {
         "filters": {
            // tridents don't normally have this family,
            // it's just an example
            "test": "is_family",
            "subject": "damager",
            "value": "trident"
         }
      },
      "damage_multiplier": 2.0
   }
}
```

Immunity to all damage except fall damage:
```json
"minecraft:damage_sensor": {
   "triggers": [
      {
         // deals_damage defaults to true
         "cause": "fall"
      },
      {
         "deals_damage": false
      }
   ]
}
```

Run an event on both the attacker and target:
```json
"components": {
   "minecraft:damage_sensor": {
      "triggers": {
         "on_damage": {
            "event": "double_event"
         }
      }
   }
},
"events": {
   "double_event": {
      "sequence": [
         {
            // could trigger another event,
            // or just do actions directly
            "trigger": {
               "event": "event1",
               "target": "self"
            }
         },
         {
            "trigger": {
               "event": "event2",
               "target": "other"
            }
         }
      ]
   }
}
```

Vanilla's way of making villagers transform into zombies:
```json
"minecraft:damage_sensor": {
   "triggers": {
      "on_damage": {
         // this is simplified somewhat
         // the original also includes a witch transformation
         "filters": [
            {
               "test": "is_family",
               "subject": "other",
               "value": "zombie"
            },
            {
               "test": "has_damage",
               "value": "fatal"
            }
         ],
         "event": "become_zombie"
      }
   }
}
```
