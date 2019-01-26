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

The `button.gamepadButtonsIndex` value must be present for the `button` to be valid.  This value maps to the index into the matching `Gamepad`'s `buttons` array.

The following are the default properties:
```json
{
    "button" : {
        "supportsTouch" : true,
        "supportsPress" : true,
        "analogValues" : false,
        "defaultValue" : 0,
        "min" : 0,
        "max" : 1,
    }
}
```

> **OPEN ISSUE** Should we have a "switch" style button?  Or is that a subtype of something else? buttons? dpads?

#### Thumbsticks
Thumbsticks always report values in a range such that `x` is between `xAxis.leftValue` and `xAxis.rightValue` and such that `y` is between `yAxis.downValue` and `yAxis.upValue`.  There is no guarantee that `xAxis.leftValue` is less than `xAxis.rightValue`, but the values will not be equal.  There is no guarantee that `yAxis.downValue` is less than `yAxis.upValue`, but the values will not be equal. 

The `xAxis.gamepadAxesIndex` and `yAxis.gamepadAxesIndex` values must be present for the `thumbstick` to be valid.  These values map to the indices into the matching `Gamepad`'s `axes` array.

A `thumbstick` may also have a `centerButton`. If so, the `centerButton.gamepadButtonsIndex` value must be present for the `thumbstick` to be valid.  This value maps to the index into the matching `Gamepad`'s `buttons` array. Additionally if `centerButton.supportsTouch` is `true`, `touched` MUST report if either `centerButton`, `xAxis`, or `yAxis` is not the `defaultValue`. 

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

If the `leftButton`, `rightButton`, `upButton`, `downButton`, or `centerButton` are present, their `gamepadButtonsIndex` values must be present for the `dpad` to be valid.  These values map to the indices into the matching `Gamepad`'s `buttons` array.

The following are the default properties:
```json
"dpad" : {
    "supportsTouch" : false,
    "leftButton" : {
        "defaultValue" : 0,
        "min" : 0,
        "max" : 1,
    },
    "rightButton" : {
        "defaultValue" : 0,
        "min" : 0,
        "max" : 1,
    },
    "upButton" : {
        "defaultValue" : 0,
        "min" : 0,
        "max" : 1,
    },
    "downButton" : {
        "defaultValue" : 0,
        "min" : 0,
        "max" : 1,
    },
    "centerButton" : {
        "defaultValue" : 0,
        "min" : 0,
        "max" : 1,
    }
}
```

#### Trackpads
Trackpads always report values in a range such that `x` is between `xAxis.leftValue` and `xAxis.rightValue` and such that `y` is between `yAxis.downValue` and `yAxis.upValue`.  There is no guarantee that `xAxis.leftValue` is less than `xAxis.rightValue`, but the values will not be equal.  There is no guarantee that `yAxis.downValue` is less than `yAxis.upValue`, but the values will not be equal. 

The `xAxis.gamepadAxesIndex` and `yAxis.gamepadAxesIndex` values must be present for the `trackpad` to be valid.  These values map to the indices into the matching `Gamepad`'s `axes` array.

A `trackpad` may also have a `dpad`. If so, the `dpad` be valid for the `trackpad` to be valid.  Additionally if `dpad.supportsTouch` is `true`, the `touched` attribute of the `leftButton`, `rightButton`, `upButton`, `downButton`, and `centerButton` MUST be `true` if a press at the current `x` and `y` position would cause the button to be pressed. 

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
        "supportsTouch" : true,
        "leftButton" : {
        },
        "rightButton" : {
        },
        "upButton" : {
        },
        "downButton" : {
        },
        "centerButton" : {
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
                "centerButton" : {
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
                "touchpadMotion" : {
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
"GearVR" : {
    "name" : "Gear VR",
    "mapping" : "xr-standard",
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
"HTCViveController" : {
    "name" : "HTC Vive Controller",
    "mapping" : "xr-standard",
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
"Knuckles" : {
    "name" : "Valve Knuckles Controller",
    "mapping" : "xr-standard",
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
                "dpad" : {
                    "centerButton" : {
                        "gamepadButtonIndex" : 0,
                    }
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
