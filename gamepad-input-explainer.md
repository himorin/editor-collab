# Gamepads in WebXR

## Overview
This explains the purpose of this document

### Notes:
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

## Schema explanation
FILL ME IN

### General
FILL ME IN

### Hands
FILL ME IN

### Components
FILL ME IN

#### Buttons
Buttons have two styles: analog or binary.  Analog buttons report analog values in a range from `min` to `max`. If the component has `supportsTouch == true`, it MUST report `touched` for `value` > `min`, but MAY also report `touched` for `value` == `min`. Binary buttons only report one of two values. If the component has `supportsTouch == true`, it MUST report `touched` for `value` == `max`, but MAY also report `touched` for `value` == `min`. 

The following are the default properties:
```json
{
    "button" : {
        "supportsTouch" : true,
        "analogValues" : false,
        "defaultValue" : 0,
        "min" : 0,
        "max" : 1,
    }
}
```

> **OPEN ISSUE** The oculus touch has a capacitive button that never fires a pressed event.... how should this be represented?

> **OPEN ISSUE** Should we have a "switch" style button?  Or is that a subtype of something else?

#### Thumbsticks
Thumbsticks always report values in a range such that `x` is between `xAxis.leftValue` and `xAxis.rightValue` and such that `y` is between `yAxis.downValue` and `yAxis.upValue`.  There is no guarantee that `xAxis.leftValue` is less than `xAxis.rightValue`, but the values will not be equal.  There is no guarantee that `yAxis.downValue` is less than `yAxis.upValue`, but the values will not be equal. A `thumbstick` may also have a `centerButton` and if it does if it does and `supportsTouch` is `true`, `touched` MUST report if either `centerButton`, `xAxis`, or `yAxis` is not the `defaultValue`. 

The following are the default properties:
```json
{
    "thumbstick" : {
        "xAxis" : {
            "defaultValue" : 0,
            "leftValue" : -1,
            "rightValue" : 1,
        },
        "yAxis" : {
            "defaultValue" : 0,
            "upValue" : -1,
            "downValue" : 1,
        },
        "centerButton" : {
            "supportsTouch" : false,
            "defaultValue" : 0,
            "min" : 0,
            "max" : 1,
        }
    }
}
```

#### Dpad
Dpads are a switch style component.  If a `leftButton` and a `rightButton` are present, only one can be pressed at a time.  If a `upButton` and a `downButton` are present, only one can be pressed at the same time.  If a `centerButton` is present, it cannot be pressed at the same time as any of the other buttons.

The following are the default properties:
```json
"dpad" : {
    "leftButton" : {
        "supportsTouch" : false,
        "defaultValue" : 0,
        "min" : 0,
        "max" : 1,
    },
    "rightButton" : {
        "supportsTouch" : false,
        "defaultValue" : 0,
        "min" : 0,
        "max" : 1,
    },
    "upButton" : {
        "supportsTouch" : false,
        "defaultValue" : 0,
        "min" : 0,
        "max" : 1,
    },
    "downButton" : {
        "supportsTouch" : false,
        "defaultValue" : 0,
        "min" : 0,
        "max" : 1,
    },
    "centerButton" : {
        "supportsTouch" : false,
        "defaultValue" : 0,
        "min" : 0,
        "max" : 1,
    }
}
```

#### Trackpads
Trackpads always report values in a range such that `x` is between `xAxis.leftValue` and `xAxis.rightValue` and such that `y` is between `yAxis.downValue` and `yAxis.upValue`.  There is no guarantee that `xAxis.leftValue` is less than `xAxis.rightValue`, but the values will not be equal.  There is no guarantee that `yAxis.downValue` is less than `yAxis.upValue`, but the values will not be equal. The `dpad` is not required to exist, nor is the `supportsTouch` property required to be true for its buttons.  However, if this is the case, the `touched` attribute of the `leftButton`, `rightButton`, `upButton`, `downButton`, and `centerButton` will be `true` if a press at the current `x` and `y` position would cause the button to be pressed. 

> **Open Issue** Are the edge buttons actually separate buttons in the buttons array?

The following are the default properties:
```json
"trackpad" : {
    "xAxis" : {
        "defaultValue" : 0,
        "leftValue" : -1,
        "rightValue" : 1,
    },
    "yAxis" : {
        "defaultValue" : 0,
        "upValue" : -1,
        "downValue" : 1,
    },
    "dpad" : {
        "leftButton" : {
            "supportsTouch" : true,
        },
        "rightButton" : {
            "supportsTouch" : true,
        },
        "upButton" : {
            "supportsTouch" : true,
        },
        "downButton" : {
            "supportsTouch" : true,
        },
        "centerButton" : {
            "supportsTouch" : true,
        }
    }
}
```
### Assets
FILL ME IN

#### Visualizations
FILL ME IN

## Known XR gamepad inventory
This section covers the mappings for all known XR controllers including those without independent tracking

### Windows Mixed Reality
```json
"045E-065D" : {
    "name" : "Windows Motion Controller",
    "mapping" : "xr-standard",
    "hands" : {
        "left" : {
            "components" : [ 0, 1, 2, 3, 4],
            "primaryButton" : 0,
            "primaryAxes" : 1
        },
        "right" : {
            "components" : [ 0, 1, 2, 3, 4],
            "primaryButton" : 0,
            "primaryAxes" : 1
        }
    },
    "components" : [
        {
            "name" : "trigger",
            "button" : {
                "gamepadButtonIndex" : 0,
                "analogValues" : true,
            }
        },
        {
            "name" : "thumbstick",
            "thumbstick" : {
                "xAxis" : {
                    "gamepadAxisIndex" : 0,
                },
                "yAxis" : {
                    "gamepadAxisIndex" : 1,
                },
                "centerButton" : {
                    "gamepadButtonIndex" : 1,
                }
            }
        },
        {
            "name" : "grip",
            "button" : {
                "gamepadButtonIndex" : 2
            }
        },
        {
            "name" : "touchpad",
            "touchpad" : {
                "xAxis" : {
                    "gamepadAxisIndex" : 2,
                },
                "yAxis" : {
                    "gamepadAxisIndex" : 3,
                },
                "dpad" : {
                    "centerButton" : {
                        "gamepadButtonIndex" : 3
                    }
                }
            }
        },
        {
            "name" : "menu",
            "button" : {
                "gamepadButtonIndex" : 4,
                "supportsTouch" : false,
            }
        },
    ],
    "assets" : {
        "leftHand" : {
            "asset" : "some uri",
            "rootNode" : "left-controller-node"
        },
        "rightHand" : {
            "asset" : "some uri",
            "rootNode" : "right-controller-node"
        },
        "visualizationNodes" : [
            {
                "component" : 0,
                "rootNode" : "trigger-node",
                "labelNode" : "trigger-label-node",
                "buttonMotion" : {
                    "target" : "trigger-transform-node",
                    "min" : "trigger-min-transform-node",
                    "max" : "trigger-max-transform-node",
                }
            },
            {
                "component" : 1,
                "rootNode" : "trigger-node",
                "labelNode" : "trigger-label-node",
                "thumbstickMotion" : {
                    "target" : "thumbstick-transform-node",
                    "left" : "thumbstick-transform-left-node",
                    "right" : "thumbstick-right-transform-right-node",
                    "up" : "thumbstick-transform-up-node",
                    "down" : "thumbstick-transform-down-node",
                    "centerMin" : "thumbstick-transform-centerMin-node",
                    "centerMax" : "thumbstick-transform-centerMax-node",
                }
            },
            {
                "component" : 2,
                "rootNode" : "grip-node",
                "labelNode" : "grip-label-node",
                "buttonMotion" : {
                    "target" : "grip-transform-node",
                    "min" : "grip-min-transform-node",
                    "max" : "grip-max-transform-node",
                }
            },
            {
                "component" : 3,
                "rootNode" : "touchpad-node",
                "labelNode" : "touchpad-label-node",
                "dpadMotion" : {
                    "target" : "touchpad-transform-node",
                    "left" : "touchpad-transform-left-node",
                    "right" : "touchpad-transform-right-node",
                    "up" : "touchpad-transform-up-node",
                    "down" : "touchpad-transform-down-node",
                    "center" : "touchpad-transform-center-node"
                },
                "touchpadMotion" : {
                    "target" : "touchpad-touchpoint-node",
                    "left" : "touchpad-touchpoint-left-node",
                    "right" : "touchpad-touchpoint-right-node",
                    "up" : "touchpad-touchpoint-up-node",
                    "down" : "touchpad-touchpoint-down-node"
                }
            },
            {
                "component" : 4,
                "rootNode" : "menu-node",
                "labelNode" : "menu-label-node",
                "buttonMotion" : {
                    "target" : "menu-transform-node",
                    "min" : "menu-min-transform-node",
                    "max" : "menu-max-transform-node",
                }
            },
        ],
    }
}
```

### Oculus Go
```json
"OculusGo" : {
    "name" : "Oculus Go",
    "mapping" : "xr-standard",
    "hands" : {
        "left" : {
            "components" : [0, 1, 2],
            "primaryButton" : 0,
            "primaryAxes" : 0
        },
        "right" : {
            "components" : [0, 1, 2],
            "primaryButton" : 0,
            "primaryAxes" : 0
        }
    },
    "components" : [
        {
            "name" : "thumbstick",
            "touchpad" : {
                "xAxis" : {
                    "gamepadAxisIndex" : 0,
                },
                "yAxis" : {
                    "gamepadAxisIndex" : 1,
                },
                "dpad" : {
                    "leftButton" : {
                        "gamepadButtonIndex" : 3,
                    },
                    "rightButton" : {
                        "gamepadButtonIndex" : 4,
                    },
                    "downButton" : {
                        "gamepadButtonIndex" : 5,
                    },
                    "upButton" : {
                        "gamepadButtonIndex" : 6,
                    },
                    "centerButton" : {
                        "gamepadButtonIndex" : 0,
                    }
                }
            }
        },
        {
            "name" : "trigger",
            "gamepadButtonIndex" : 1,
            "analogValues" : true,
        },
        {
            "name" : "faceButton",
            "gamepadButtonIndex" : 2,
        },
    ],
    "assets" : {
        "leftHand" : {
            "asset" : "some uri",
            "rootNode" : "neutral-controller-node"
        },
        "rightHand" : {
            "asset" : "some uri",
            "rootNode" : "neutral-controller-node"
        },
        "visualizationNodes" : [
            {
                "component" : 0,
                "rootNode" : "touchpad-node",
                "labelNode" : "touchpad-label-node",
                "dpadMotion" : {
                    "target" : "touchpad-transform-node",
                    "left" : "touchpad-transform-left-node",
                    "right" : "touchpad-transform-right-node",
                    "up" : "touchpad-transform-up-node",
                    "down" : "touchpad-transform-down-node",
                    "center" : "touchpad-transform-center-node"
                },
                "touchpadMotion" : {
                    "target" : "touchpad-touchpoint-node",
                    "left" : "touchpad-touchpoint-left-node",
                    "right" : "touchpad-touchpoint-right-node",
                    "up" : "touchpad-touchpoint-up-node",
                    "down" : "touchpad-touchpoint-down-node"
                }
            },
            {
                "component" : 1,
                "rootNode" : "trigger-node",
                "labelNode" : "trigger-label-node",
                "buttonMotion" : {
                    "target" : "trigger-transform-node",
                    "min" : "trigger-min-transform-node",
                    "max" : "trigger-max-transform-node",
                }
            },
            {
                "component" : 1,
                "rootNode" : "faceButton-node",
                "labelNode" : "faceButton-label-node",
                "buttonMotion" : {
                    "target" : "faceButton-transform-node",
                    "min" : "faceButton-min-transform-node",
                    "max" : "faceButton-max-transform-node",
                }
            },
        ],
    }
}
```

### Oculus Touch
```json
"OculusTouch" : {
    "name" : "Oculus Touch",
    "mapping" : "xr-standard",
    "components" : [
        {
            "name" : "thumbstick",
            "buttonsIndex" : 0,
            "nodeName" : "thumbstick-model-node",
            "analog" : false,
            "clickable" : true,
            "touchable" : true,
            "xAxis" : { // I don't actually see this documented anywhere?
                "axisIndex" : 0,
                "analog" : true,
                "left" : 0,
                "right" : 1
            },
            "yAxis" : {
                "axisIndex" : 1,
                "analog" : true,
                "down" : 0,
                "up" : 1
            }
        },
        {
            "name" : "trigger",
            "buttonsIndex" : 1,
            "nodeName" : "trigger-model-node",
            "analog" : true,
            "clickable" : true,
            "touchable" : true,
        },
        {
            "name" : "grip",
            "buttonsIndex" : 2,
            "nodeName" : "grip-model-node",
            "analog" : false,
            "clickable" : true,
            "touchable" : false,
        },
        {
            "name" : "x",
            "buttonsIndex" : 3,
            "nodeName" : "x-model-node",
            "analog" : false,
            "clickable" : true,
            "touchable" : true,
        },
        {
            "name" : "y",
            "buttonsIndex" : 4,
            "nodeName" : "y-model-node",
            "analog" : false,
            "clickable" : true,
            "touchable" : true,
        },
        {
            "name" : "a",
            "buttonsIndex" : 3,
            "nodeName" : "a-model-node",
            "analog" : false,
            "clickable" : true,
            "touchable" : true,
        },
        {
            "name" : "b",
            "buttonsIndex" : 4,
            "nodeName" : "b-model-node",
            "analog" : false,
            "clickable" : true,
            "touchable" : true,
        },
        { // Couldn't find this one documented anywhere
            "name" : "thumbrest",
            "buttonsIndex" : 5,
            "analog" : false,
            "touchable" : true,
            "clickable" : false,
        },
    ],
    "assets" : [
        "Some uri"
    ],
    "hands" : {
        "left" : {
            "components" : [0, 1, 2, 3, 4, 7],
            "primary" : [0, 1],
            "assetId" : 0,
            "nodeName" : "left",
        },
        "right" : {
            "components" : [0, 1, 2, 5, 6, 7],
            "primary" : [0, 1],
            "assetId" : 0,
            "nodeName" : "right",
        }
    }
}
```

### Gear VR
```json
"GearVR" : {
    "name" : "Gear VR",
    "mapping" : "xr-standard",
    "components" : [
        {
            "name" : "thumbstick",
            "buttonsIndex" : 0,
            "nodeName" : "thumbstick-model-node",
            "analog" : false,
            "clickable" : true,
            "touchable" : true,
            "xAxis" : {
                "axisIndex" : 0,
                "analog" : false,
                "left" : -1,
                "right" : 1
            },
            "yAxis" : {
                "axisIndex" : 1,
                "analog" : false,
                "down" : -1,
                "up" : 1
            }
        },
        {
            "name" : "faceButton",
            "buttonsIndex" : 1,
            "nodeName" : "faceButton-model-node",
            "analog" : false,
            "clickable" : true,
            "touchable" : false,
        },
    ],
    "assets" : [
        "Some uri"
    ],
    "hands" : {
        "neutral" : {
            "components" : [0, 1],
            "primary" : [0],
            "assetId" : 0,
            "nodeName" : "neutral",
        }
    }
}
```

### Oculus Remote
```json
"OculusRemote" : { 
    "name" : "Oculus Remote",
    "mapping" : "",
    "components" : [
        {
            "name" : "button",
            "buttonsIndex" : 0,
            "analog" : false,
            "clickable" : true,
            "touchable" : false,
        },
        {
            "name" : "thumbstick",
            "buttonsIndex" : 0,
            "analog" : false,
            "clickable" : false,
            "touchable" : false,
            "xAxis" : {
                "axisIndex" : 0,
                "analog" : false,
                "left" : -1,
                "right" : 1
            },
            "yAxis" : {
                "axisIndex" : 1,
                "analog" : false,
                "down" : -1,
                "up" : 1
            }
        },
        {
            "name" : "back",
            "buttonsIndex" : 1,
            "analog" : false,
            "clickable" : true,
            "touchable" : false,
        },
    ],
    "assets" : [
        "Some uri"
    ],
    "hands" : {
        "neutral" : {
            "components" : [0, 1],
            "primary" : [0],
            "assetId" : 0,
            "nodeName" : "neutral",
        }
    }
}
```

### HTC Vive
```json
"HTCViveController" : {
    "name" : "HTC Vive Controller",
    "mapping" : "xr-standard",
    "components" : [
        {
            "name" : "trackpad",
            "buttonsIndex" : 0,
            "nodeName" : "trackpad-model-node",
            "analog" : true,
            "clickable" : true,
            "touchable" : true,
            "xAxis" : {
                "axisIndex" : 0,
                "analog" : true,
                "left" : -1,
                "right" : 1
            },
            "yAxis" : {
                "axisIndex" : 1,
                "analog" : true,
                "down" : -1,
                "up" : 1
            }
        },
        {
            "name" : "trigger",
            "buttonsIndex" : 1,
            "clickable" : true,
            "touchable" : true,
            "analog" : true,
        },
        {
            "name" : "grip",
            "buttonsIndex" : 2,
            "analog" : false,
            "touchable" : false,
            "clickable" : true,
        },
    ],
    "assets" : [
        "Some uri"
    ],
    "hands" : {
        "neutral" : {
            "components" : [0, 1, 2],
            "primary" : [1],
            "assetId" : 0,
            "nodeName" : "neutral",
        }
    }
}
```

### Valve Knuckles
```json
"Knuckles" : {
    "name" : "Valve Knuckles Controller",
    "mapping" : "xr-standard",
    "components" : [
        {
            "name" : "trackpad",
            "buttonsIndex" : 0,
            "analog" : true,
            "clickable" : true,
            "touchable" : true,
            "xAxis" : {
                "axisIndex" : 0,
                "analog" : true,
                "left" : -1,
                "right" : 1
            },
            "yAxis" : {
                "axisIndex" : 1,
                "analog" : true,
                "down" : -1,
                "up" : 1
            }
        },
        {
            "name" : "trigger",
            "buttonsIndex" : 1,
            "clickable" : true,
            "touchable" : true,
            "analog" : true,
        },
        {
            "name" : "innerFaceButton",
            "buttonsIndex" : 2,
            "analog" : false,
            "touchable" : false,
            "clickable" : true,
        },
        {
            "name" : "outerFaceButton",
            "buttonsIndex" : 3,
            "analog" : false,
            "touchable" : false,
            "clickable" : true,
        },
    ],
    "assets" : [
        "Some uri"
    ],
    "hands" : {
        "left" : {
            "components" : [0, 1, 2, 3],
            "primary" : [1],
            "assetId" : 0,
            "nodeName" : "left",
        },
        "right" : {
            "components" : [0, 1, 2, 3],
            "primary" : [1],
            "assetId" : 0,
            "nodeName" : "right",
        }
    }
}
```

### Vive Focus
FILL ME IN

### Magic Leap
FILL ME IN

### Daydream
FILL ME IN

### Mirage Solo
FILL ME IN

### HoloLens Clicker
Should this be included?  Should the Oculus Remote be removed?

## Appendices

### References
* [GitHub - stewdio/THREE.VRController: Support hand controllers for Oculus, Vive, Windows Mixed Reality, Daydream, GearVR, and more by adding VRController to your existing Three.js-based WebVR project.](https://github.com/stewdio/THREE.VRController)
* [assets/controllers at gh-pages · aframevr/assets · GitHub](https://github.com/aframevr/assets/tree/gh-pages/controllers)
* [Unity - Manual:  Input for OpenVR controllers](https://docs.unity3d.com/Manual/OpenVRControllers.html)
* [Steam VR Template -        Unreal Engine Forums](https://forums.unrealengine.com/development-discussion/vr-ar-development/78620-steam-vr-template?106609-Steam-VR-Template=)
* [Mapping Oculus Controller Input to Blueprint Events](https://developer.oculus.com/documentation/unreal/latest/concepts/unreal-controller-input-mapping-reference/)
