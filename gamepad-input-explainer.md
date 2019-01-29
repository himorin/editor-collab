# Gamepads in WebXR
This proposal goes in tandem with [PR #26](https://github.com/immersive-web/editor-collab/pull/26).

On the Web, the [`Gamepad` API](https://www.w3.org/TR/gamepad/) is used to provide access to multi-button input devices that aren't mice.

## Overview
This explains the purpose of this document

When working with an `XRInputSource` with a `gamepad` member, it's often important to 

### Notes/questions that drove design
* Model to render
* Buttons
* Types of buttons (trigger, grip, touchpad. thumbstick, etc)
* Axes vs. not? (Need to be very clear which way they go)
  * Is there a button associated with the axes?
* priority ordering of buttons (which touchpad/thumbstick matters)
* Where is each button relative to the gripSpace?
  * for putting a “how do you use me” label
* Does the button move? 
  * Is there bone ID?
  * What are the directions/limits of motion?
  * Touchpad location indicator?
* Still to poke at 
  * Oculus Go and Touch fires events at the extents of the motion controller bounds.
  * Vive seems to also treat trackpad regions as "being buttons"
  * figure out how to represent trackpad edge clicks

# Schema explanation
FILL ME IN
```json
{
    "version" : "",
    "gamepad" : [],
    "user-agent-overrides" : []
}
```

## Gamepads
FILL ME IN
```json
{
    "gamepad" : {
        "id" : "<Gamepad object's ID>",
        "hands" : {},
        "dataSources" : [],
        "components" : [],
        "responses" : []
    }
}
```

### Hands
For motion controllers that report a handedness property
```json
"hands" : {
    "left" : {},
    "right" : {}
}
```

For motion controllers than can't distinguish right vs left
```json
"hands" : {
    "neutral" : {}
}
```

The description of the hands data goes HERE
```json
{
    "asset" : "",
    "rootId" : "<id of the model's root node in asset",
    "componentIds" : [],
    "primaryButton" : #,
    "primaryAxes" : #
}
```

#### Components
```json
"components" : [
    {
        "dataSource" : #,
        "rootId" : "<id of component's root node in the model>",
        "labelTransformId" : "<id of the component's label transform in asset>",
        "pressResponse" : #,
        "touchResponse" : #
    }
]
```

### Data Sources
EXPLAIN ME

```json
"dataSources" : [
    {
        "button" : {},
        "dpad" : {},
        "thumbstick" : {},
        "touchpad" : {}
    }
]
```

#### Buttons
Buttons have two styles: analog or binary.  Analog buttons report analog values in a range from `min` to `max`. If the data source has `supportsTouch == true`, it MUST report `touched` for `value` > `min`, but MAY also report `touched` for `value` == `min`. Binary buttons only report one of two values. If the data source has `supportsTouch == true`, it MUST report `touched` for `value` == `max`, but MAY also report `touched` for `value` == `min`. 

The `button.gamepadButtonsIndex` value must be present for the `button` to be valid.  This value maps to the index into the matching `Gamepad`'s `buttons` array.

The following are the default properties:

```json
"dataSources" : [
    {
        "button" : {
            "id" : "",
            "supportsTouch" : true,
            "supportsPress" : true,
            "analogValues" : false,
            "defaultValue" : 0,
            "min" : 0,
            "max" : 1,
            "gamepadButtonsIndex" : <index in gamepad's buttons array>
        }
    }
]
```

#### Dpad
Dpads are a switch style data source.  If a left button and a right button are present, only one can be pressed at a time.  If an up button and a down button are present, only one can be pressed at the same time.

The following are the default properties:
```json
"dataSources" : [
    {
        "dpad" : {
            "id" : "",
            "supportsTouch" : false,
            "supportsPress" : true,
            "analogValues" : false,
            "defaultValue" : 0,
            "min" : 0,
            "max" : 1,
            "gamepadButtonsIndices" : {
                "left" : <index in gamepad's buttons array>,
                "right" : <index in gamepad's buttons array>,
                "up" : <index in gamepad's buttons array>, 
                "down" : <index in gamepad's buttons array>
            }
        }
    }
]
```

#### Thumbsticks
Thumbsticks always report values in a range such that `x` is between `xAxis.leftValue` and `xAxis.rightValue` and such that `y` is between `yAxis.downValue` and `yAxis.upValue`.  There is no guarantee that `xAxis.leftValue` is less than `xAxis.rightValue`, but the values will not be equal.  There is no guarantee that `yAxis.downValue` is less than `yAxis.upValue`, but the values will not be equal. 

The `xAxis.gamepadAxesIndex` and `yAxis.gamepadAxesIndex` values must be present for the `thumbstick` to be valid.  These values map to the indices into the matching `Gamepad`'s `axes` array.

A `thumbstick` may also have a `button`. If so, the `button.gamepadButtonsIndex` value must be present for the `thumbstick` to be valid.  This value maps to the index into the matching `Gamepad`'s `buttons` array. Additionally if `button.supportsTouch` is changed to `true`, the button's `touched` MUST report if either `button`, `xAxis`, or `yAxis` is not the `defaultValue`. 

The following are the default properties:
```json
"dataSources" : [
    {
        "thumbstick" : {
            "id" : "",
            "xAxis" : {
                "defaultValue" : 0,
                "leftValue" : -1,
                "rightValue" : 1,
                "gamepadAxesIndex" : <index in gamepad's axes array>
            },
            "yAxis" : {
                "defaultValue" : 0,
                "upValue" : -1,
                "downValue" : 1,
                "gamepadAxesIndex" : <index in gamepad's axes array>
            },
            "button" : {
                "supportsTouch" : false,
                "supportsPress" : true,
                "analogValues" : false,
                "defaultValue" : 0,
                "min" : 0,
                "max" : 1,
                "gamepadButtonsIndex" : <index in gamepad's buttons array>
            }
        }
    }
]
```

#### Touchpad
Touchpads always report values in a range such that `x` is between `xAxis.leftValue` and `xAxis.rightValue` and such that `y` is between `yAxis.downValue` and `yAxis.upValue`.  There is no guarantee that `xAxis.leftValue` is less than `xAxis.rightValue`, but the values will not be equal.  There is no guarantee that `yAxis.downValue` is less than `yAxis.upValue`, but the values will not be equal. 

The `xAxis.gamepadAxesIndex` and `yAxis.gamepadAxesIndex` values must be present for the `touchpad` to be valid.  These values map to the indices into the matching `Gamepad`'s `axes` array.

A `touchpad` may also have a `button`. If so, the `button` must be valid for the `touchpad` to be valid.  Additionally if `button.supportsTouch` is set to `true`, the `touched` attribute of the `button` MUST be `true` if `x` or `y` are not their default values.

The following are the default properties:
```json
"dataSources" : [
    {
        "touchpad" : {
            "id" : "",
            "xAxis" : {
                "defaultValue" : 0,
                "leftValue" : -1,
                "rightValue" : 1,
                "gamepadAxesIndex" : <index in gamepad's axes array>
            },
            "yAxis" : {
                "defaultValue" : 0,
                "upValue" : -1,
                "downValue" : 1,
                "gamepadAxesIndex" : <index in gamepad's axes array>
            },
            "button" : {
                "supportsTouch" : false,
                "supportsPress" : true,
                "analogValues" : false,
                "defaultValue" : 0,
                "min" : 0,
                "max" : 1,
                "gamepadButtonsIndex" : <index in gamepad's buttons array>
            }
        }
    }
]
```

### Responses
FILL ME IN
```json
"responses" : [
    {
        "buttonPress" : {},
        "buttonTouch" : {},
        "dpadPress" : {},
        "thumbstickTouch" : {},
        "touchpadPress" : {},
        "touchpadTouch" : {}
    }
]
```

FILL ME IN
```json
"buttonPress" : {
    "target" : "<element to be modified>",
    "default" : "<the button's default visual state>",
    "min" : "<the button's minimum visual state>",
    "max" : "<the button's maximum visual state>",
}
```

FILL ME IN
```json
"dpadPress" : {
    "target" : "<element to be modified>",
    "default" : "<the dpad's default visual state>",
    "left" : "<the dpad's left button's maximum visual state>",
    "right" : "<the dpad's right button's maximum visual state>",
    "down" : "<the dpad's down button's maximum visual state>",
    "up" : "<the dpad's up button's maximum visual state>"
}
```

FILL ME IN
```json
"thumbstickTouch" : {
    "target" : "<element to be modified>",
    "default" : "<the thumbstick's default visual state>",
    "left" : "<the thumbstick's xAxis's left-most visual state>",
    "right" : "<the thumbstick's xAxis's right-most visual state>",
    "down" : "<the thumbstick's yAxis's down-most visual state>",
    "up" : "<the thumbstick's yAxis's down-most visual state>",
    "min" : "<the thumbstick's button's minimum visual state>",
    "max" : "<the thumbstick's button's maximum visual state>",
}
```

FILL ME IN
```json
"touchpadPress" : {
    "target" : "<element to be modified>",
    "default" : "<the touchpad's default visual state>",
    "left" : "<the visual state of a press on the touchpad's left>",
    "right" : "<the visual state of a press on the touchpad's right>",
    "down" : "<the visual state of a press on the touchpad's down>",
    "up" : "<the visual state of a press on the touchpad's up>"
}
```

FILL ME IN
```json
"touchpadTouch" : {
    "target" : "<element to be modified>",
    "default" : "<the touchpad's default visual state>",
    "left" : "<the visual state of a touch on the touchpad's left>",
    "right" : "<the visual state of a touch on the touchpad's right>",
    "down" : "<the visual state of a touch on the touchpad's down>",
    "up" : "<the visual state of a touch on the touchpad's up>"
}
```

## User-agent overrides
FILL ME IN - and design me

# Appendices

### Additional Hardware
* Vive Focus
* Magic Leap
* Daydream
* Mirage Solo
* HoloLens Clicker (Should this be included?)
* Oculus Remote (Should this be included?)

### References
* [GitHub - stewdio/THREE.VRController: Support hand controllers for Oculus, Vive, Windows Mixed Reality, Daydream, GearVR, and more by adding VRController to your existing Three.js-based WebVR project.](https://github.com/stewdio/THREE.VRController)
* [assets/controllers at gh-pages · aframevr/assets · GitHub](https://github.com/aframevr/assets/tree/gh-pages/controllers)
* [Unity - Manual:  Input for OpenVR controllers](https://docs.unity3d.com/Manual/OpenVRControllers.html)
* [Steam VR Template -        Unreal Engine Forums](https://forums.unrealengine.com/development-discussion/vr-ar-development/78620-steam-vr-template?106609-Steam-VR-Template=)
* [Mapping Oculus Controller Input to Blueprint Events](https://developer.oculus.com/documentation/unreal/latest/concepts/unreal-controller-input-mapping-reference/)
* [Events in the Gamepad spec?](https://github.com/w3c/gamepad/pull/15)
