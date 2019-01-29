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
        "components" : [],
        "assets" : {}
    }
}
```

### Hands
For motion controllers that report a handedness property
```json
"hands" : {
    "left" : {
        "components" : [#],
        "primaryButton" : #,
        "primaryAxes" : #
    },
    "right" : {
        "components" : [#],
        "primaryButton" : #,
        "primaryAxes" : #
    }
}
```

For motion controllers than can't distinguish right vs left
```json
"hands" : {
    "neutral" : {
        "components" : [#],
        "primaryButton" : #,
        "primaryAxes" : #
    }
}
```

### Components
FILL ME IN

#### Buttons
Buttons have two styles: analog or binary.  Analog buttons report analog values in a range from `min` to `max`. If the component has `supportsTouch == true`, it MUST report `touched` for `value` > `min`, but MAY also report `touched` for `value` == `min`. Binary buttons only report one of two values. If the component has `supportsTouch == true`, it MUST report `touched` for `value` == `max`, but MAY also report `touched` for `value` == `min`. 

The `button.gamepadButtonsIndex` value must be present for the `button` to be valid.  This value maps to the index into the matching `Gamepad`'s `buttons` array.

The following are the default properties:
```json
"components" : [
    {
        "button" : {
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
Dpads are a switch style component.  If a left button and a right button are present, only one can be pressed at a time.  If an up button and a down button are present, only one can be pressed at the same time.

The following are the default properties:
```json
"components" : [
    {
        "dpad" : {
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
"components" : [
    {
        "thumbstick" : {
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
"components" : [
    {
        "trackpad" : {
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
### Assets
FILL ME IN
Note that for this section, the two rootNodes could also be the same
```json
"assets" : {
    "hands" : {
        "left" : {},
        "right" : {},
        "neutral" : {}
    },
    "mappings" : [],
    "motions" : []
}
```
OR

```json
"hand" : {
    "asset" : "",
    "rootId" : "<id of the model's root node in asset",
    "components" : [
        {
            "componentId" : #,
            "rootId" : "<id of the component's root node in asset>",
            "labelTransformId" : "<id of the component's label transform in asset>",
            "motion" : #
        }
    ]
}
```

#### Motion types
FILL ME IN
```json
"motions" : [
    {
        "buttonMotion" : {},
        "dpadMotion" : {},
        "thumbstickMotion" : {},
        "touchpadMotion" : {}
    }
]
```

FILL ME IN
```json
"buttonMotion" : {
    "target" : "<button transform id>"

}
```

FILL ME IN
```json
"dpadMotion" : {}
```

FILL ME IN
```json
"thumbstickMotion" : {}
```

FILL ME IN
```json
"touchpadMotion" : {}
```

## User-agent overrides
FILL ME IN - and design me

# Known XR gamepad inventory
This section covers the mappings for all known XR controllers

### Windows Mixed Reality
```json
"gamepad" : {
    "name" : "045E-065D", // Windows Motion Controller
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
                "button" : {
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
                "button" : {
                    "gamepadButtonIndex" : 3
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
                "rootNode" : "thumbstick-node",
                "labelNode" : "thumbstick-label-node",
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
                    "centerMin" : "touchpad-transform-centerMin-node",
                    "centerMax" : "touchpad-transform-centerMax-node"
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
"gamepad" : {
    "name" : "Oculus-Go",
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
                "button" : {
                    "gamepadButtonIndex" : 0,
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
"gamepad" : {
    "name" : "Oculus-Touch",
    "hands" : {
        "left" : {
            "components" : [0, 1, 2, 3, 4, 7],
            "primaryButton" : 0,
            "primaryAxes" : 1
        },
        "right" : {
            "components" : [0, 1, 2, 5, 6, 7],
            "primaryButton" : 0,
            "primaryAxes" : 1
        }
    },
    "components" : [
        {
            "name" : "thumbstick",
            "thumbstick" : {
                "xAxis" : { // I don't actually see this documented anywhere?
                    "gamepadAxisIndex" : 0,
                    "left" : 0,
                    "right" : 1
                },
                "yAxis" : {
                    "gamepadAxisIndex" : 1,
                    "down" : 0,
                    "up" : 1
                },
                "button" : {
                    "gamepadButtonIndex" : 0,
                    // Is this touchable?
                }
            }
        },
        {
            "name" : "trigger",
            "button" : {
                "gamepadButtonIndex" : 1,
                "analogValues" : true,
            }
        },
        {
            "name" : "grip",
            "button" : {
                "gamepadButtonIndex" : 2,
                "supportsTouch" : false,
            }
        },
        {
            "name" : "x",
            "button" : {
                "gamepadButtonIndex" : 3,
                "supportsTouch" : false,
            }
        },
        {
            "name" : "y",
            "button" : {
                "gamepadButtonIndex" : 4,
                "supportsTouch" : false,
            }
        },
        {
            "name" : "a",
            "button" : {
                "gamepadButtonIndex" : 3,
                "supportsTouch" : false,
            }
        },
        {
            "name" : "b",
            "button" : {
                "gamepadButtonIndex" : 4,
                "supportsTouch" : false,
            }
        },
        { // Couldn't find this one documented anywhere
            "name" : "thumbrest",
            "button" : {
                "gamepadButtonIndex" : 5,
                "supportsPress" : false,
            }
        }
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
                "rootNode" : "thumbstick-node",
                "labelNode" : "thumbstick-label-node",
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
                "rootNode" : "x-node",
                "labelNode" : "x-label-node",
                "buttonMotion" : {
                    "target" : "x-transform-node",
                    "min" : "x-min-transform-node",
                    "max" : "x-max-transform-node",
                }
            },
            {
                "component" : 4,
                "rootNode" : "y-node",
                "labelNode" : "y-label-node",
                "buttonMotion" : {
                    "target" : "y-transform-node",
                    "min" : "y-min-transform-node",
                    "max" : "y-max-transform-node",
                }
            },
            {
                "component" : 5,
                "rootNode" : "a-node",
                "labelNode" : "a-label-node",
                "buttonMotion" : {
                    "target" : "a-transform-node",
                    "min" : "a-min-transform-node",
                    "max" : "a-max-transform-node",
                }
            },
            {
                "component" : 6,
                "rootNode" : "b-node",
                "labelNode" : "b-label-node",
                "buttonMotion" : {
                    "target" : "b-transform-node",
                    "min" : "b-min-transform-node",
                    "max" : "b-max-transform-node",
                }
            },
            {
                "component" : 7,
                "rootNode" : "thumbrest-node",
                "labelNode" : "thumbrest-label-node",
                "buttonMotion" : {
                    "target" : "thumbrest-touchpoint-node",
                    "min" : "thumbrest-min-touchpoint-node",
                    "max" : "thumbrest-max-touchpoint-node",
                }
            },
        ],
    }
}
```

### Gear VR
```json
"gamepad" : {
    "name" : "Gear-VR",
    "hands" : {
        "neutral" : {
            "components" : [0, 1, 2],
            "primaryButton" : 0,
            "primaryAxes" : 0
        },
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
                "button" : {
                    "gamepadButtonIndex" : 0,
                }
            }
        },
        {
            "name" : "faceButton",
            "gamepadButtonIndex" : 2,
            "supportsTouch" : false
        },
    ],
    "assets" : {
        "neutralHand" : {
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
                    "centerMin" : "touchpad-transform-centerMin-node",
                    "centerMax" : "touchpad-transform-centerMax-node"
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
                "rootNode" : "faceButton-node",
                "labelNode" : "faceButton-label-node",
                "buttonMotion" : {
                    "target" : "faceButton-transform-node",
                    "min" : "faceButton-min-transform-node",
                    "max" : "faceButton-max-transform-node",
                }
            }
        ]
    }
}
```

### HTC Vive
```json
"gamepad" : {
    "id" : "HTC-Vive-Controller",
    "hands" : {
        "neutral" : {
            "components" : [ 0, 1, 2],
            "primaryButton" : 0,
            "primaryAxes" : 0
        },
    },
    "components" : [
        {
            "name" : "trackpad",
            "touchpad" : {
                "xAxis" : {
                    "gamepadAxisIndex" : 0,
                },
                "yAxis" : {
                    "gamepadAxisIndex" : 1,
                },
                "button" : {
                    "gamepadButtonIndex" : 0,
                }
            }
        },
        {
            "name" : "trigger",
            "gamepadButtonIndex" : 1,
            "analogValues" : true,
        },
        {
            "name" : "grip",
            "gamepadButtonIndex" : 2,
            "supportsTouch" : false
        },
    ],
    "assets" : {
        "neutralHand" : {
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
                    "centerMin" : "touchpad-transform-centerMin-node",
                    "centerMax" : "touchpad-transform-centerMax-node"
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
                "rootNode" : "grip-node",
                "labelNode" : "grip-label-node",
                "buttonMotion" : {
                    "target" : "grip-transform-node",
                    "min" : "grip-min-transform-node",
                    "max" : "grip-max-transform-node",
                }
            },
        ],
    }
}
```

### Valve Knuckles
```json
"gamepad" : {
    "id" : "Knuckles",
    "hands" : {
        "left" : {
            "components" : [0, 1, 2, 3],
            "primaryButton" : 0,
            "primaryAxes" : 0
        },
        "right" : {
            "components" : [0, 1, 2, 3],
            "primaryButton" : 0,
            "primaryAxes" : 0
        }
    },
    "components" : [
        {
            "name" : "trackpad",
            "touchpad" : {
                "xAxis" : {
                    "gamepadAxisIndex" : 0,
                },
                "yAxis" : {
                    "gamepadAxisIndex" : 1,
                },
                "centerButton" : {
                    "gamepadButtonIndex" : 0,
                }
            }
        },
        {
            "name" : "trigger",
            "button" : {
                "gamepadButtonIndex" : 1,
                "analogValues" : true,
            }
        },
        {
            "name" : "innerFaceButton",
            "gamepadButtonIndex" : 2,
            "supportsTouch" : false
        },
        {
            "name" : "outerFaceButton",
            "gamepadButtonIndex" : 3,
            "supportsTouch" : false
        }
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
                "rootNode" : "touchpad-node",
                "labelNode" : "touchpad-label-node",
                "dpadMotion" : {
                    "target" : "touchpad-transform-node",
                    "centerMin" : "touchpad-transform-centerMin-node",
                    "centerMax" : "touchpad-transform-centerMax-node"
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
                "component" : 2,
                "rootNode" : "innerFaceButton-node",
                "labelNode" : "innerFaceButton-label-node",
                "buttonMotion" : {
                    "target" : "innerFaceButton-transform-node",
                    "min" : "innerFaceButton-min-transform-node",
                    "max" : "innerFaceButton-max-transform-node",
                }
            },
            {
                "component" : 3,
                "rootNode" : "outerFaceButton-node",
                "labelNode" : "outerFaceButton-label-node",
                "buttonMotion" : {
                    "target" : "outerFaceButton-transform-node",
                    "min" : "outerFaceButton-min-transform-node",
                    "max" : "outerFaceButton-max-transform-node",
                }
            }
        ],
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
Should this be included?  

### Oculus Remote
Should this be included?  

## Appendices

### References
* [GitHub - stewdio/THREE.VRController: Support hand controllers for Oculus, Vive, Windows Mixed Reality, Daydream, GearVR, and more by adding VRController to your existing Three.js-based WebVR project.](https://github.com/stewdio/THREE.VRController)
* [assets/controllers at gh-pages · aframevr/assets · GitHub](https://github.com/aframevr/assets/tree/gh-pages/controllers)
* [Unity - Manual:  Input for OpenVR controllers](https://docs.unity3d.com/Manual/OpenVRControllers.html)
* [Steam VR Template -        Unreal Engine Forums](https://forums.unrealengine.com/development-discussion/vr-ar-development/78620-steam-vr-template?106609-Steam-VR-Template=)
* [Mapping Oculus Controller Input to Blueprint Events](https://developer.oculus.com/documentation/unreal/latest/concepts/unreal-controller-input-mapping-reference/)
* [Events in the Gamepad spec?](https://github.com/w3c/gamepad/pull/15)
