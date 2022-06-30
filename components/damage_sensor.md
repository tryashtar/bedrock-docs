## `minecraft:damage_sensor`

This component is used to detect, modify, or suppress damage dealt to an entity.

It consists of one or more damage triggers. When the entity is dealt damage, the first trigger that matches the damage will take effect. It could run an [entity event](../events.md), change the amount of damage taken, or even prevent it altogether.

After one trigger activates, any subsequent triggers are ignored, even if they would otherwise match the damage.

---

## JSON Structure

The component has only one field: <img src="../icons/object.png" width=16>/<img src="../icons/list.png" width=16> `triggers`. It can either be a single trigger object, or a list of them. A trigger object has several possible fields, all of them optional.

By default, a trigger will activate for *all* kinds of damage. There are two fields that allow for more specific control:

|Type|Name|Description|
|-|-|-|
|<img src="../icons/string.png" width=16>|`cause`|Only activate for [damage of a specific type](../damage-types.md).|
|<img src="../icons/object.png" width=16> <img src="../icons/list.png" width=16>|`filters`|Only activate when the [filter](../filters.md) passes. This goes inside an `on_damage` object alongside `event`. The `has_damage` test is often used here to check for `fatal` damage.|

There is no way to check for the *amount* of damage. If the checks pass, the trigger will activate. There are three fields that can modify the damage in response:

|Type|Name|Description|
|-|-|-|
|<img src="../icons/bool.png" width=16>|`deals_damage`|Defaults to true. When set to false, the damage is completely canceled.|
|<img src="../icons/int.png" width=16>|`damage_modifier`|Added to the damage taken. Can be made negative to reduce taken damage, but can't reduce it below 0.|
|<img src="../icons/float.png" width=16>|`damage_multiplier`|Multiplies the damage taken, after `damage_modifier`. Can't reduce it below 1, unless `damage_modifier` already did.|

Lastly, there are three fields that can be used to perform some action in response to the damage:

|Type|Name|Description|
|-|-|-|
|<img src="../icons/string.png" width=16>|`event`|Name of an [entity event](../events.md) to run. This goes inside an `on_damage` object alongside `filters`.|
|<img src="../icons/string.png" width=16>|`target`|Subject for the event. This also goes inside `on_damage`, next to `event`.|
|<img src="../icons/string.png" width=16>|`on_damage_sound_event`|[Sound event](../sound-events.md) to play. This plays in addition to the entity's normal hurt sound. It can either be an entry in `individual_event_sounds`, or the entity's own `entity_sounds`.|

In `filters`, there are three different entities that can be used as the `subject`:
* `self` is the entity taking damage, like normal.
* `damager` is the entity dealing the damage.
* `other` is the entity responsible for the damage.

For ranged attacks, `damager` is the projectile, and `other` is the shooter. For melee attacks, or projectiles not shot by an entity, `damager` and `other` are both the same entity. For damage not caused by an entity, neither of these are set, so any filters using them will fail.

These entities can also be used as the event `target`, [except for `damager`](https://bugs.mojang.com/browse/MCPE-157652).

---

## Additional Information and Bugs
* The official docs say that `damage_modifier` is a float. This is effectively false; any value provided after a decimal point is ignored.

* Reducing damage with `damage_modifier` and `damage_multiplier` can cause problems with the damage invulnerability cooldown.
  * Whenever an entity *receives* damage and gets hurt (typically turning red and making a noise), the game saves the damage amount.
  * For the next second, if any incoming damage is *less* than this amount, it will be ignored. If any incoming damage is *greater* than the amount, the difference between the two will be taken instead.
  * When reducing damage with these fields, the incoming damage will always be greater than the received damage. Therefore, the entity will be eligible for getting hurt every tick, but will ultimately take no damage.
  * <p><video src="../videos/damage_modifier.mp4" width="400"/></p>

* The damage from `/kill` cannot be detected or prevented. `/kill` bypasses damage sensors entirely.
* Damage sensors can both detect and produce zero-damage hurts. For example, the resistance V effect causes entities to take zero damage from all sources, but still turn red and make noise. Setting `deals_damage` to false will prevent this.
* Fire damage will not activate damage sensors if the entity is immune due to the fire resistance effect or [`fire_immune`](./fire_immune.md) component.
* `fall` damage is unique: when reduced to zero by any means, the entity will not get hurt (turn red and make noise). This was most likely added as a special case so goats and frogs didn't look like they were taking fall damage from large jumps. Damage sensors can still detect zero-damage fall damage. Also note that the feather falling enchantment can never reduce fall damage below one point of damage.

* [MCPE-155588](https://bugs.mojang.com/browse/MCPE-155588): An invalid damage type in `cause` is silently ignored.
* [MCPE-155589](https://bugs.mojang.com/browse/MCPE-155589): Damage from blocks like magma and berry bushes sets the `other` subject equal to the `self` subject, and activates triggers even in creative mode.
* [MCPE-108988](https://bugs.mojang.com/browse/MCPE-108988): A `has_damage` filter test for `fatal` damage will pass when taking damage from blocks like magma and berry bushes, regardless of whether the damage was fatal.
* [MCPE-66473](https://bugs.mojang.com/browse/MCPE-66473): A `has_damage` filter test for `fatal` damage will pass if the base damage is enough to kill the entity, not taking into account armor reduction.
* [MCPE-157652](https://bugs.mojang.com/browse/MCPE-157652): `damager` doesn't work as an event's `target`.

---

## Examples
Take double damage from tridents:
```jsonc
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

Run a command when punched:
```jsonc
"components": {
   "minecraft:damage_sensor": {
      "triggers": {
         "cause": "entity_attack",
         "on_damage": {
            "filters": {
               "test": "is_family",
               "subject": "other",
               "value": "player"
            },
            "event": "punched"
         },
         "deals_damage": false
      }
   }
},
"events": {
   "punched": {
      "run_command": {
         "command": "scoreboard players add @s test 1"
      }
   }
}
```

Immunity to all damage except fall damage:
```jsonc
"minecraft:damage_sensor": {
   "triggers": [
      {
         // deals_damage defaults to true,
         // so we don't have to specify it
         "cause": "fall"
      },
      {
         "deals_damage": false
      }
   ]
}
```

Run an event on both the attacker and target:
```jsonc
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
```jsonc
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
