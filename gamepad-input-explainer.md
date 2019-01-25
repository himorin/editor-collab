# Gamepads in WebXR

## Notes:
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

#### References:
* [GitHub - stewdio/THREE.VRController: Support hand controllers for Oculus, Vive, Windows Mixed Reality, Daydream, GearVR, and more by adding VRController to your existing Three.js-based WebVR project.](https://github.com/stewdio/THREE.VRController)
* [assets/controllers at gh-pages · aframevr/assets · GitHub](https://github.com/aframevr/assets/tree/gh-pages/controllers)
* [Unity - Manual:  Input for OpenVR controllers](https://docs.unity3d.com/Manual/OpenVRControllers.html)
* [Steam VR Template -        Unreal Engine Forums](https://forums.unrealengine.com/development-discussion/vr-ar-development/78620-steam-vr-template?106609-Steam-VR-Template=)

> I'm assuming that we'll normalize all the axes and values.  Values go from [0, 1], axes go from [-.5 to .5]

```json
{
    "gamepads" : {
        "045E-065D" : {
            "name" : "Microsoft Motion Controller",
            "mapping" : "xr-standard",
            "assets" : [
                "Some uri"
            ],
            "hands" : {
                "left" : {
                    "components" : [ 0, 1, 2, 3, 4],
                    "primary" : [0, 1],
                    "asset" : 0,
                    "nodeName" : "left",
                },
                "right" : {
                    "components" : [ 0, 1, 2, 3, 4],
                    "primary" : [0, 1],
                    "asset" : 0,
                    "nodeName" : "right",
                }
            },
            "components" : [
                {
                    "name" : "trigger",
                    "button" : {
                        "buttonsIndex" : 0,
                        "analog" : true,
                        "clickable" : true,
                        "touchable" : false,
                        "min" : 0, 
                        "max" : 1,
                    }
                },
                {
                    "name" : "thumbstick",
                    "button" : {
                        "buttonsIndex" : 1,
                        "analog" : false,
                        "clickable" : true,
                        "touchable" : false,
                        "min" : 0, 
                        "max" : 1,
                    },
                    "xAxis" : {
                        "axesIndex" : 0,
                        "analog" : true,
                        "clickable" : true,
                        "touchable" : false,
                        "left" : -1,
                        "right" : 1,
                    },
                    "yAxis" : {
                        "axesIndex" : 1,
                        "analog" : true,
                        "down" : -1,
                        "up" : 1
                    }
                },
                {
                    "name" : "grip",
                    "button" : {
                        "buttonsIndex" : 2,
                        "analog" : true,
                        "clickable" : true,
                        "touchable" : false,
                        "min" : 0, 
                        "max" : 1,
                    }
                },
                {
                    "name" : "touchpad",
                    "button" : {
                        "buttonsIndex" : 3,
                        "analog" : true,
                        "clickable" : true,
                        "touchable" : true,
                        "min" : 0, 
                        "max" : 1,
                    },
                    "xAxis" : {
                        "axisIndex" : 2,
                        "analog" : true,
                        "clickable" : true,
                        "touchable" : true,
                        "left" : -1,
                        "right" : 1
                    },
                    "yAxis" : {
                        "axisIndex" : 3,
                        "analog" : true,
                        "clickable" : true,
                        "touchable" : true,
                        "down" : -1,
                        "up" : 1
                    }
                },
                {
                    "name" : "menu",
                    "button" : {
                        "buttonsIndex" : 4,
                        "analog" : false,
                        "clickable" : true,
                        "touchable" : false,
                        "min" : 0, 
                        "max" : 1,
                    }
                },
            ],
            "animations" : [
                {
                    "component" : 0,
                    "nodeName" : "trigger-node",
                    "button" : {
                        "minNodeName" : "trigger-node",
                        "maxNodeName" : "trigger-max-node",
                    }
                },
                {
                    "component" : 1,
                    "nodeName" : "thumbstick-node",
                    "button" : {
                        "minNodeName" : "thumbstick-node",
                        "maxNodeName" : "thumbstick-max-node",
                    },
                    "xAxis" : {
                        "leftNodeName" : "thumbstick-left-node",
                        "rightNodeName" : "thumbstick-right-node"
                    },
                    "yAxis" : {
                        "upNodeName" : "thumbstick-up-node",
                        "downNodeName" : "thumbstick-down-node"
                    }
                },
                {
                    "component" : 1,
                    "nodeName" : "touchpad-touch-node",
                    "xAxis" : {
                        "leftNodeName" : "touchpad-touch-left-node",
                        "rightNodeName" : "touchpad-touch-right-node"
                    },
                    "yAxis" : {
                        "upNodeName" : "touchpad-touch-up-node",
                        "downNodeName" : "touchpad-touch-down-node"
                    }
                }
            ]
        },


        "OculusGo" : {
            "name" : "Oculus Go",
            "mapping" : "xr-standard",
            "components" : [
                {
                    "name" : "thumbstick",
                    "button" : {
                        "buttonsIndex" : 0,
                        "analog" : false,
                        "clickable" : true,
                        "touchable" : true,
                    },
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
                    "name" : "trigger",
                    "buttonsIndex" : 1,
                    "nodeName" : "trigger-model-node",
                    "analog" : true,
                    "clickable" : true,
                    "touchable" : false,
                },
                {
                    "name" : "faceButton",
                    "buttonsIndex" : 2,
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
                "left" : {
                    "components" : [0, 1, 2],
                    "primary" : [0],
                    "assetId" : 0,
                    "nodeName" : "neutral",
                },
                "right" : {
                    "components" : [0, 1, 2],
                    "primary" : [0],
                    "assetId" : 0,
                    "nodeName" : "neutral",
                }
            }
        },


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
        },


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
        },


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
        },


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
        },


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
        },


        /*
            Missing:
            * Vive Focus
            * Magic Leap
            * Daydream
            * Mirage Solo
            * HoloLens clicker
        */ 
    }
}
```