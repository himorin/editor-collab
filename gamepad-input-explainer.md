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
  * Figure out how to add the joint/transform details

#### References:
* [GitHub - stewdio/THREE.VRController: Support hand controllers for Oculus, Vive, Windows Mixed Reality, Daydream, GearVR, and more by adding VRController to your existing Three.js-based WebVR project.](https://github.com/stewdio/THREE.VRController)
* [assets/controllers at gh-pages · aframevr/assets · GitHub](https://github.com/aframevr/assets/tree/gh-pages/controllers)
* [Unity - Manual:  Input for OpenVR controllers](https://docs.unity3d.com/Manual/OpenVRControllers.html)
* [Steam VR Template -        Unreal Engine Forums](https://forums.unrealengine.com/development-discussion/vr-ar-development/78620-steam-vr-template?106609-Steam-VR-Template=)


```json
{
    "gamepads" : {
        "045E-065D" : {
            "name" : "Microsoft Motion Controller",
            "mapping" : "xr-standard",
            "components" : [
                {
                    "name" : "trigger",
                    "buttonIndex" : 0,
                    "analog" : true,
                    "clickable" : true,
                    "touchable" : false,
                },
                {
                    "name" : "thumbstick",
                    "buttonIndex" : 1,
                    "analog" : true,
                    "clickable" : true,
                    "touchable" : false,
                    "xAxis" : {
                        "axisIndex" : 0,
                        "analog" : true,
                        "left" : -1,
                        "right" : 1
                    },
                    "yAxis" : {
                        "axisIndex" : 1,
                        "analog" : true,
                        "down" : 1,
                        "up" : -1
                    }
                },
                {
                    "name" : "grip",
                    "buttonIndex" : 2,
                    "analog" : true,
                    "clickable" : true,
                    "touchable" : false,
                },
                {
                    "name" : "trackpad",
                    "buttonIndex" : 3,
                    "analog" : true,
                    "clickable" : true,
                    "touchable" : true,
                    "xAxis" : {
                        "axisIndex" : 2,
                        "analog" : true,
                        "left" : -1,
                        "right" : 1
                    },
                    "yAxis" : {
                        "axisIndex" : 3,
                        "analog" : true,
                        "down" : -1,
                        "up" : 1
                    }
                },
                {
                    "name" : "menu",
                    "buttonIndex" : 4,
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
                    "components" : [0, 1, 2, 3, 4],
                    "primary" : [0, 1],
                    "assetId" : 0,
                    "nodeName" : "left",
                },
                "right" : {
                    "components" : [0, 1, 2, 3, 4],
                    "primary" : [0, 1],
                    "assetId" : 0,
                    "nodeName" : "right",
                }
            }
        },


        "OculusGo" : {
            "name" : "Oculus Go",
            "mapping" : "xr-standard",
            "components" : [
                {
                    "name" : "thumbstick",
                    "buttonIndex" : 0,
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
                    "name" : "trigger",
                    "buttonIndex" : 1,
                    "analog" : true,
                    "clickable" : true,
                    "touchable" : false,
                },
                {
                    "name" : "faceButton",
                    "buttonIndex" : 2,
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
                    "buttonIndex" : 0,
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
                    "buttonIndex" : 1,
                    "analog" : true,
                    "clickable" : true,
                    "touchable" : true,
                },
                {
                    "name" : "grip",
                    "buttonIndex" : 2,
                    "analog" : false,
                    "clickable" : true,
                    "touchable" : false,
                },
                {
                    "name" : "x",
                    "buttonIndex" : 3,
                    "analog" : false,
                    "clickable" : true,
                    "touchable" : true,
                },
                {
                    "name" : "y",
                    "buttonIndex" : 4,
                    "analog" : false,
                    "clickable" : true,
                    "touchable" : true,
                },
                {
                    "name" : "a",
                    "buttonIndex" : 3,
                    "analog" : false,
                    "clickable" : true,
                    "touchable" : true,
                },
                {
                    "name" : "b",
                    "buttonIndex" : 4,
                    "analog" : false,
                    "clickable" : true,
                    "touchable" : true,
                },
                { // Couldn't find this one documented anywhere
                    "name" : "thumbrest",
                    "buttonIndex" : 5,
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
                    "buttonIndex" : 0,
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
                    "buttonIndex" : 1,
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
                    "buttonIndex" : 0,
                    "analog" : false,
                    "clickable" : true,
                    "touchable" : false,
                },
                {
                    "name" : "thumbstick",
                    "buttonIndex" : 0,
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
                    "buttonIndex" : 1,
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
                    "buttonIndex" : 0,
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
                    "buttonIndex" : 1,
                    "clickable" : true,
                    "touchable" : true,
                    "analog" : true,
                },
                {
                    "name" : "grip",
                    "buttonIndex" : 2,
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
                    "buttonIndex" : 0,
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
                    "buttonIndex" : 1,
                    "clickable" : true,
                    "touchable" : true,
                    "analog" : true,
                },
                {
                    "name" : "innerFaceButton",
                    "buttonIndex" : 2,
                    "analog" : false,
                    "touchable" : false,
                    "clickable" : true,
                },
                {
                    "name" : "outerFaceButton",
                    "buttonIndex" : 3,
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