# WebXR Device API - Input
This document is a subsection of the main WebXR Device API explainer document which can be found [here](explainer.md).  The main explainer contains all the information you could possibly want to know about setting up a WebXR session, the render loop, and more.  In contrast, this document covers how to manage input across the range of XR hardware.

## Concepts
XR hardware provides a wide variety of input mechanisms including, but not limited to, screen taps, spatially-tracked articulated hands, single button clickers, voice commands, and fully spatially-tracked controllers with multiple buttons, joysticks, triggers, or touchpads.

The goal of the WebXR API is to enable the widest-range of potential input devices, which it does by focusing on what they have in common and categorizing them based on their differences.  The commonality is that users will always need a way to aim in 3D space and a mechanism to indicate a when they are attempting to interact with what they are aiming at.  This concept is the foundation of the `XRInputSource` design and is known as "aim and select".

### Aiming categories
The `XRInputSource` inherits from `XRSpace` and the intended aiming ray is defined as the forward vector in this space. While the selection mechanism will vary uniquely for each input device, the aiming mechanisms can be grouped into three categories: `tracked-pointer`, `gaze`, and `screen`.  

#### Tracked Pointer
Tracked pointers are input sources that are tracked independently of the viewer's location.  Examples include the Oculus Touch motion controllers and the Magic Leap hand tracking.

In addition to the `XRSpace` intrinsic to an `XRInputSource`, this category of `XRInputSource` also exposes a secondary `XRSpace` called the `gripSpace` that communicates the location of the motion controller in a user's hand.  This different from the aiming ray, which often will have an origin at the tip of the motion controller and be angled slightly downward for comfort. The exact orientation of the ray relative to a given device should follow platform-specific guidelines if there are any. In the absence of platform-specific guidance or a physical device, the target ray should most likely point in the same direction as the user's index finger if it was outstretched.  For more details this topic, see the [Grip space](#Grip-space) section.

#### Gaze
Gaze-based input sources do not have their own tracking mechanism and instead use the viewer's head position for aiming. Example include 0DOF clickers, headset buttons, regular gamepads, and certain voice commands. Within this category, some input sources are persistent (e.g. those backed by hardware) while others will come-and-go when invoked by the user (e.g. voice commands).

#### Screen
When using an XR session on a 2D screen, pointer events on the relevant screen regions are monitored and `XRInputSource`s are generated in response to allow unified input handling with immersive mode controller or gaze input. For inline sessions with an `outputContext`, the monitored region is the `outputContext`'s canvas.  For immersive sessions (e.g. hand-held AR), the entire screen is monitored. The screen-based `XRInputSource`'s aiming ray must originate at the point that was interacted with on the canvas, projected onto the near clipping plane (defined by the `depthNear` attribute of the `XRSession`) and extending out into the scene along that projected vector.

### Selection styles
In addition to an aiming ray, all `XRInputSource` objects must provide a mechanism for the user to perform a "select" action. This user intent is communicated to developers through events which are discussed in detail in the [Input events](#Input-events) section. The physical action which triggers this selection differs based on the input type, and may indicate (but is not limited to):

  * Pressing a trigger
  * Clicking a touchpad
  * Tapping a button
  * Making a hand gesture
  * Speaking a command
  * Clicking or touching the screen

### Emulate poses
Some input sources may not be able to accurately track their position in space, and instead provide an estimated position based on the sensor data available. This is the case, for example, for the Daydream and GearVR 3DoF controllers, which use an arm model to approximate controller position based on rotation. It may also be true on devices while under tracking loss.  In these situations, the `emulatedPosition` attribute of the `XRPose` returned by `XRSession.getPose()` must be set to `true` to indicate that the translation components of retrieved pose matrices may not be accurate.

## Enumerating input sources
Calling the `getInputSources()` function on an `XRSession` will return a list of all `XRInputSource`s that the user agent considers active. 
The properties of an `XRInputSource` object are immutable. If a device can be manipulated in such a way that these properties can change, the `XRInputSource` will be removed and recreated.

```js
let inputSources = xrSession.getInputSources();
```

When input sources are added to or removed from the list of available input sources the `inputsourceschange` event must be fired on the `XRSession` object to indicate that any cached copies of the list should be refreshed.  This event is of the type `XRInputSourceChangeEvent` and will contain three attributes: `session` is associated session being changed, `added` is the new input sources, and `removed` is the input sources that will no longer be reported. In addition, the `inputsourceschange` event will also fire once after the session creation callback completes.

```js
function onSessionStarted(session) {
  // Session initialization logic ...

  xrSession.addEventListener('inputsourceschange', onInputSourcesChange);

  // More session initialization logic ...
}

let xrInputSources = null;
function onInputSourcesChange(event) {
  xrInputSources = event.session.getInputSources();
}
```

## Input events
When the selection mechanism for an `XRInputSource` is invoked, three `XRInputSourceEvent` events are fired on the `XRSession`.

* `selectstart` indicates an action has been initiated. It will most commonly be associated with pressing a button or trigger.
* `selectend` indicates an action has ended. It will most commonly be associated with releasing a button or trigger. A `selectend` event must also be fired if the input source is disconnected after an action has been initiated, or the action has otherwise been cancelled. In that case an associated `select` event will not be fired.
* `select` indicates that a action has been completed. A `select` event is considered to be [triggered by user activation](https://html.spec.whatwg.org/multipage/interaction.html#triggered-by-user-activation)

The `selectstart` and `selectend` events are useful for handling dragging, painting, or other continuous motions. As the `select` event is trigger by user activation, it can be used to begin playing media or other trusted interactions.

```js
function onSessionStarted(session) {
  // Session initialization logic ...

  session.addEventListener("select", onSelect);
  session.addEventListener("selectstart", onSelectStart);
  session.addEventListener("selectend", onSelectEnd);

  // More session initialization logic ...
}
```
The event's `inputSource` attribute must contain the `XRInputSource` that produced the event. The event's `frame` attribute must contain a valid `XRFrame` that can be used to query the input and device poses at the time the selection event occurred. The `XRViewerPose`'s `views` array must be empty.

>**QUESTION** now that we have a viewerSpace.... can we just have the getViewerPose call fail when not during a render frame?

### Transient input sources
Some input sources are only be added to the list of input sources while an action is occurring. For example, those with an `inputMode` of `screen` or those with `inputMode` of `gaze` that are triggered by a voice command. In these cases, `XRInputSource` is only present in the array returned by `getInputSources()` during the lifetime of the action. In this circumstance, the order of events is as follows:
1. Optionally `pointerdown`
1. `inputsourceschange` for add
1. `selectstart`
1. Optionally `pointermove`
1. `selectend`
1. Optionally `click`
1. `select`
1. `inputsourceschange` for remove
1. Optionally `pointerup`

Input sources that are instantaneous, without a clear start and end point such as a verbal command like "Select", must still fire all events in the sequence.  All input sources present for an `XRFrame` must inherit from a meaningful `XRSpace` and calls to `getPose()` must behave normally.

### Choosing a preferred input source
Many platforms support multiple input sources concurrently. Examples of this are left/right handed motion controllers or hand tracking combined with a 0DOF clicker. Since `xrSession.getInputSources()` returns all connected input sources, application may choose to take into consideration the most recently used input sources when rendering UI hints, such as a cursor, ray or highlight.

To simplify the sample code throughout this explainer, an example is provided which shows one potential way to set a `preferredInputSource`.

```js
// Keep track of the preferred input source
var preferredInputSource = null;

function onSelectStart(event) {
    // Update the preferred input source to be the last one the user interacted with
    preferredInputSource = event.inputSource;
}

function onInputSourceChanged(event) {
  xrInputSources = event.session.getInputSources();

  // Choose an appropriate default from available inputSources, such as 
  // prioritizing based on the value of inputMode: 'screen' over 
  // 'tracked-pointer' over 'gaze'.
  preferredInputSource = computePreferredInputSource();
}
```

## Hit testing
"Hit testing" (aka "raycasting") is the process of finding intersections between 3D geometry and a ray, comprised of an origin and direction. Hit testing can be done against virtual 3D geometry or real-world 3D geometry. Most commonly in WebXR, developers will hit test from `XRInputSource`s or from the `XRSession.viewerSpace`.  Some purposes of hit testing might be to determine user intention, track where a cursor should be drawn on hand-held devices, or even to bounce a virtual object off real-world geometry. In WebXR, 'inline' and 'immersive-vr' sessions are limited to performing virtual hit tests, while 'immersive-ar' sessions can perform both virtual and real-world hit tests. 

### Virtual hit testing
A virtual hit test is a hit test that returns results from intersections with virtual 3D geometry. WebXR does not have any knowledge of the developer's 3D scene graph, but does have information about the real-world location of `XRInputSource` objects.  Using the `XRFrame.getPose()` function, developers can create an `XRRay` representing the `origin` and `direction` of the user's intent. The `origin` represents a 3D coordinate in space with a `w` component that must be 1, and the `direction` represents a normalized 3D directional vector with a `w` component that must be 0. Both values are given as `DOMPointReadOnly`s. The `XRRay`'s `matrix` represents the transform from a ray originating at `[0, 0, 0]` and extending down the negative Z axis to the ray described by the `XRRay`'s `origin` and `direction`. This is useful for positioning graphical representations of the ray.

```js
function updateScene(timestamp, xrFrame) {
  // Scene update logic ...

  let hitTestSourcePose = frame.getPose(preferredInputSource, xrReferenceSpace);
  if (hitTestSourcePose) {
    // Invoke the example 3D engine to compute the virtual hit test
    var virtualHitTestResult = scene.virtualHitTest(new XRRay(hitTestSourcePose.transform));
  }

  // Other scene update logic ...
}
```

### Real-world hit testing
A key challenge with enabling real-world hit testing in WebXR is that computing real-world hit test results can be performance-impacting and dependant on secondary threads in many of the underlying implementations.  However from a developer perspective, out-of-date asynchronous hit test results are often, though not always, less than useful. 

WebXR addresses this challenge through the use of the `XRHitTestSource` type which acts somewhat like a subscription mechanism. The presence of an `XRHitTestSource` signals to the user agent that the developer intends to query `XRHitTestResult`s in subsequent `XRFrame`s.  The user agent can then precompute `XRHitTestResult`s based on the `XRHitTestSource` properties such that each `XRFrame` will be bundled with all "subscribed" hit test results. When the last reference to the `XRHitTestResult` has been released, the user agent is free to stop computing `XRHitTestResult`s for future frames.

#### Requesting a hit test source
To create an `XRHitTestSource` developers call the `XRSession.requestHitTestSource()` function.  This function accepts an `XRHitTestOptionsInit` dictionary with the following values:
* The `space` is required and is the `XRSpace` to be tracked by the hit test source.
* The `offsetRay` is optional and is the `XRRay` from which the hit test should be performed. This ray is defined relative to the `space`. If an `offsetRay` is not provided, the default is an `XRRay` with a forward direction and an origin coincident with the `space` origin.

>**QUESTION** Should we leave out the `featureType` for the first PR?  If so, I'll need to change the grab-and-drag sample.

>**TODO** figure out how to know hand-held vs. head-worn and show this sample based on that

In this example, an `XRHitTestSource` is created slightly above the center of the `viewerSpace`. This is because the developer has chosen to leave space for buttons along the bottom of the hand-held AR device screen while still giving the perception of a centered cursor (see [Rendering cursors and highlights](#Rendering-cursors-and-highlights)

```js
let viewerHitTestSource = null;

let hitTestOptions = { space:xrSession.viewerSpace, offsetRay:new XRRay({ origin: {y: 0.5} }) };
xrSession.requestHitTestSource(hitTestOptions).then((hitTestSource) => {
  viewerHitTestSource = hitTestSource;
});
```

#### Automatic hit test source creation
While asynchronous hit test source creation is useful for a number of scenarios, it is problematic for [transient input sources](#Transient-Input-Sources).  If an `XRHitTestSource` is requested in response to the `inputsourceschange` event, it may be several frames before the new hit test source is able to provide hit test results. By which time the input source may no longer exist.

To mitigate this difficulty, for each new `XRInputSource` user agents must automatically generate an `XRHitTestSource` to be retrievable by the `XRInputSourcesChangeEvent.getHitTestSource()` function.  When passed an `XRInputSource` from the `XRInputSourcesChangeEvent.added` attribute, the pre-generated `XRHitTestSource` will be returned.  Any other parameter will return throw a `DOMException`. It's worth noting that, as mentioned above, each active `XRHitTestSource` may have a negative performance impact.  Therefore instead of an implicit binding of `XRHitTestSource` objects to the lifetime of `XRInputSource` objects, developers must explicitly choose to retain the `XRHitTestSource` if they wish to use it. Otherwise, at the end of the `inputsourceschange` callbacks the pre-created hit test source will disappear.

```js
let hitTestSources = {};

function onInputSourcesChange(event) {
  xrInputSources = event.session.getInputSources();

  foreach (inputSource of event.removed) {
    delete hitTestSources[inputSource];
  }

  foreach (inputSource of event.added)
    hitTestSources[inputSource] = event.getHitTestSource(inputSource);
  }

  updatePreferredInputSource();
}
```

#### Hit test results
Using an `XRHitTestSource`, the developer can get synchronous hit test results on each frame. This is done by 

```js
function updateScene(timestamp, xrFrame) {
  // Scene update logic ...

  let hitTestResults = xrFrame.getHitTestResults(hitTestSources[preferredInputSource], xrReferenceSpace);

  // Other scene update logic ...
}
```

On occasion, developers may want hit test results for the current frame even if they have not already created an `XRHitTestSource` to subscribe to the results. For example, when a virtual object needs to bounce off a real-world surface, a single hit-test result can be requested.  The result will be delivered asynchronously, though it will be accurate for the frame on which it was requested.

```js
function updateScene(timestamp, xrFrame) {
  // Scene update logic ...

  xrFrame.requestHitTestResults(hitTestSources[preferredInputSource], xrReferenceSpace).then((results) => {
    // do something with results
  });

  // Other scene update logic ...
}
```

### Combining virtual and real-world hit testing
A key component to creating realistic presence in XR experiences, relies on the ability to know if a hit test intersects virtual or real-world geometry first. For example, a virtual button should be "clicked" if the user selects it, but not if there's a physical object in the way. There are a handful to different techniques which can be used to determine a combined hit test result that take into account a user's previous actions.  For example, a developer may choose to weight hit test results differently if a user is already interacting with a particular object. 

For the purposes of this explainer, a simple example of combining hit test results is provided below.  This sample compares the first real-world hit test result (real-world hit-test results are returned closest to farthest in the array) to the virtual hit test result and returns the one closest to the origin of the hit-test request.

```js
function updateScene(timestamp, xrFrame) {
  // Scene update logic ...

  let hitTestResult = getHitCombinedHitTestResult(xrFrame);

  // Other scene update logic ...
}

function getHitCombinedHitTestResult(frame, inputSource, hitTestSource) {
  let combinedResult = {};

  let viewerPose = frame.getViewerPose(xrReferenceSpace);
  if (!viewerPose)
    return;

  if (hitTestSource) {
    var realHitTestResults = frame.getHitTestResults(hitTestSource, xrReferenceSpace);
  }

  if (inputSource) {
    let inputSourcePose = frame.getPose(inputSource.source, xrReferenceSpace);
    if (inputSourcePose) {
      var virtualHitTestResult = scene.virtualHitTest(new XRRay(inputSource.transform));
    }
  }

  // Explain that this is a grossly simplified way of determining which hit test result to use
  let isVirtualCloser = false;
  if (virtualHitTestResult && realHitTestResults[0]) {
    let virtualDistance = MatrixMathLibrary.distance(viewerPose.transform, virtualHitTestResult.transform);
    let realDistance = MatrixMathLibrary.distance(viewerPose.transform, realHitTestResults[0].transform);
    isVirtualCloser = (virtualDistance < realDistance);
  } else if (virtualHitTestResult) {
    isVirtualCloser = true;
  }
  
  combinedResult["result"] = (isVirtualCloser) ? virtualHitTestResult : realHitTestResults[0];
  combinedResult["virtualTarget"] = (isVirtualCloser) ? virtualHitTestResult.target : null;

  return combinedResult;
}
```

### Grab-and-Drag
Another common operation in XR experiences is grabbing a virtual object and moving it to a new location. There are a number of ways to accomplish this experience, and a simple example is provided below to illustrate on approach to doing so with the WebXR apis.

The first step in the grab-and-drag operation is the "grab" step and is done in response to the `selectstart` event. If a draggable virtual object is hit by the `preferredInputSource`, the drag operation is begun by saving the data necessary for the next steps and giving the object a highlight that indicates it is being dragged.  Additionally, a new `XRHitTestSource` is created based on the `preferredInputSource` to ensure that the draggable object is only placed on horizontal real-world surfaces.

```js
let activeDragInteraction = null;

function onSelectStart(event) {
  // Select start logic ...

  // Ignore the event if we are already dragging
  if (!activeDragInteraction) {
    
    // Update the preferred input source to be the last one the user interacted with
    preferredInputSource = event.inputSource;

    // Use the input's hitTestSource to find a draggable object in the scene
    let combinedHitTestResult = getHitCombinedHitTestResult(event.frame, 
                                                            preferredInputSource, 
                                                            hitTestSources[preferredInputSource]);

    // The virtualTarget object isn't part of the WebXR API.  It is
    // something set by the imaginary 3D engine in this example
    let virtualTarget = combinedHitTestResult["virtualTarget"];

    if (virtualTarget && virtualTarget.isDraggable) {
      activeDragInteraction = {
        inputSource: inputSource,
        target: virtualTarget,
        initialTargetTransform: virtualTarget.transform,
        initialHitTestResult: combinedHitTestResult["result"]
      };

      // Change the hit-testing to look for a horizontal plane
      let hitTestOptions = { space:preferredInputSource, featureType:"horizontal-plane" };
      xrSession.requestHitTestSource(hitTestOptions).then((hitTestSource) => {
        activeDragInteraction.hitTestSource = hitTestSource;
      });

      // Use imaginary 3D engine to indicate active drag object
      scene.addDragIndication(activeDragInteraction.target);
    }
  }

  // Other select start logic ...
}
```

On each frame, the location of the virtual object being dragged is updated so that it slides along virtual and real-world geometry as the user aims with their input source.

```js
function updateScene() {
  // Scene update logic ...

  if (activeDragInteraction) {
    let inputSource = activeDragInteraction.inputSource;
    let hitTestSource = activeDragInteraction.hitTestSource;
    let combinedHitTestResult = getHitCombinedHitTestResult(event.frame, inputSource, hitTestSource);
    if (combinedHitTestResult["result"]) {
      activeDragInteraction.target.setTransform(combinedHitTestResult.transform);
    }
  }

  // Other scene update logic ...
}
```

When the user releases the "select" gesture, the drag event is completed and the sample double checks that the virtual object will fit at the new location.

>**TODO** This is where I'll update to put the anchor sample when we're ready to include it

```js
function onSelectEnd(event) {
  // Only end the drag when the input source that started dragging releases the select action
  if (activeDragInteraction && event.inputSource == activeDragInteraction.inputSource) {

    let combinedHitTestResult = getHitCombinedHitTestResult(event.frame, 
                                                            activeDragInteraction.inputSource, 
                                                            activeDragInteraction.hitTestSource);
    if (combinedHitTestResult["result"]) {
      let target = activeDragInteraction.target;
      let result = combinedHitTestResult["result"];

      // Check if the object being dragged can fit at the requested 
      // destination by comparing the bounds to the imaginary 3D engine's 
      // target.footprint 
      if (MatrixMathLibrary.contains(result.bounds, target.footprint)) {
        target.setTransform(result.transform);
      } else {
        target.setTransform(initialTargetTransform.transform);
      }
    }

    activeDragInteraction = null;
  }
}
```

## Rendering Input
When rendering an XR scene, it is often useful for users to have visual representations of their input sources and targeting hints.  The mechanisims for displaying this information are often categorized as follows:
* **Cursor** indicates the intersection with virtual or real geometry that an `XRHitTestSource` is aimed at
* **Highlight** changed properties of a virtual object to indicate a hit test source it aimed at it
* **Virtual model** a virtual representation of a physical `XRInputSource`
* **Pointing ray** a ray from the origin of an `XRHitTestSource.space` that terminates at the intersection with virtual or real geometry

The appropriateness of using these visualizations depends on various factors, including the value of the input source's `inputMode`.  

|                   | Cursor | Highlight | Virtual Model   | Pointing Ray |
| ------------------| ------ | --------- | --------------  | ------------ |
| `gaze`            | √      | √         | X               | X            |
| `tracked-pointer` | √      | √         | √ (if possible) | √            |
| `screen`          | X      | √         | X               | X            |

As indicated in the table above, `gaze` style input sources should not draw a pointing ray.  This is because the ray's origin would be located between the user's eyes and may obscure the user's vision or be difficult to visually converge on.  Additionally, `screen` style inputs should only use highlights as the user's fingers will obscure  other visualizations. Lastly, developers should only attempt to render a virtual model for `tracked-pointer` input sources.  For further explanation, see the [Loading renderable models](#Loading-renderable-models) section.

```js
function updateScene(timestamp, xrFrame) {
  // Scene update logic ...

  let combinedHitTestResult = getHitCombinedHitTestResult(xrFrame,
                                                          preferredInputSource,
                                                          hitTestSources[preferredInputSource]);

  let hitTestResult = combinedHitTestResult["result"];
  updateCursor(hitTestResult);
  updateHighlight(hitTestResult);
  updateVirtualInputModels();
  updatePointingRay(hitTestResult);
  
  // Other scene update logic ...
}
```

### Cursors and highlights
In the sample code below, the cursor is turned on for all non-screen based input sources and positioned based on the hit-test result.  If the hit test result is a virtual object, a highlight is added to the object. This sample also disables these visualization during drag operations as the object being dragged would obscure the cursor and a different highlight is used.  

```js
function updateCursor(hitTestResult) {
  // Toggle the cursor in the imaginary 3D engine
  if (!hitTestResult || preferredInputSource.inputMode == "screen" || activeDragInteraction) {
    scene.cursor.visible = false;
  } else {
    scene.cursor.visible = true;
    scene.cursor.setTransform(hitTestResult.transform);
  }
}

function updateTargetHighlight(hitTestResult) {
  // The virtualTarget object isn't part of the WebXR API.  It is
  // something set by the imaginary 3D engine in this example
  if (hitTestResult && hitTestResult.virtualTarget && !activeDragInteraction) {
    scene.addTargetIndication(hitTestResult.virtualTarget);
  }
}
```

### Loading renderable models
For `tracked-controller` input sources, it is often appropriate for the application to render a contextually appropriate model (such as a racket in a tennis game), in which case the physical appearance of the input source doesn't matter much. Many other times, though, it's desirable for the application to display a device that matches what the user is holding, especially when relaying instructions about it's use. Finally there are times when it's best to not render anything at all, such as when the XR device uses a transparent display and the user can see their hands and/or any tracked devices without app assistance. See [Handling non-opaque displays](explainer.md#Handling-non-opaque-displays) in the main explainer for more details.

The `XRInputSource.renderId` is used to determine what should be rendered in the cases when an application specific model isn't being used.

* A `renderId` of '' (empty string) indicates the input source should not be rendered. Cursors and pointing rays should still be rendered as described above if needed by the application. Input sources with a `inputMode` of 'gaze' or 'screen' MUST have a renderId of ''. 'tracked-pointer' input sources should typically only have a renderId of '' when using displays that allow the user to see their environment and the physical input source.
* The `renderId` may be 'unknown' if the type of input source cannot be reliably identified or the UA determines that the input source type must be masked for any reason.
* Otherwise the renderId should be a lower-case string that describes the physical input source. It should not include and indication of the handedness of the input source, as that should be implied by the handedness attribute.
  * If the input source is a tracked hand without any held controller, the renderId MUST be 'hand'
  * For most controllers this should ideally be of the format <vendor>-<product-id>. For example: oculus-touch. UAs should make an effort to align on the strings that are returned for any given device.
  * TODO: A canonical list of known input devices and their established `renderId`s should be created within this repo.
* Inline sessions MUST only expose renderIds of '' or 'unknown'.

The WebXR Device API currently does not offer any way to retrieve renderable resources that represent the input devices from the API itself, and as such the `renderId` must be used as a key to load an appropriate resources from the application's server or a CDN. The example below presumes that the sample `getControllerMesh()` call would do the required lookup.

```js
function onInputSourceChanged(event) {
  // Input source changed logic

  // Imaginary 3D engine clears 3D objects for removed XRInputSources
  foreach(inputSource of event.removed) {
    scene.removeInputObject(inputSource);
  }

  // Imaginary 3D engine adds a new 3D object for the input source
  foreach(inputSource of event.added) {
    if (inputSource.inputMode == "tracked-pointer" && xrSession.environmentBlendMode == "opaque") {
      scene.addInputObject(inputSource, getControllerMesh(inputSource.renderId));
    }
  }
  
  // More input source changed logic ...
}
```

### Grip space
A virtual model can only be rendered if the input source has a non-null `gripSpace`.  The `gripSpace` is an `XRSpace` where, if the user was holding a straight rod in their hand, it would be aligned with the negative Z axis (forward) and the origin rests at their palm. This enables developers to properly render a virtual object held in the user's hand. For example, a sword would be positioned so that the blade points directly down the negative Z axis and the center of the handle is at the origin. As mentioned in the [Tracked Pointer](#Tracked-pointer) section, the `gripSpace` is not the same as the `XRSpace` intrinsic to the `XRInputSource`.  The `gripSpace` will be `null` if the input source isn't trackable.

On each frame the pose associated with the `gripSpace` should be retrieved by calling `XRFrame.getPose()` and providing the `XRSpace` the pose should be reported in.  Typically, this will be the `XRReferenceSpace` used to retrieve the `XRViewerPose`, but it is not restricted as such. Just like `getViewerPose()`, `getPose()` may return `null` in cases where tracking has been lost, or the `XRSpace`'s `XRInputSource` instance is no longer connected or available.

```js
function updateVirtualInputSources() {
  // Scene update logic ...

  foreach(inputSource of scene.inputObjects.keys) {
    if(inputSource.gripSource) {
      let pose = frame.getPose(inputSource.gripSpace, xrReferenceFrame);
      let inputObject = scene.inputSources[inputSource];
      if (pose) {
        inputObject.setTransform(pose.transform);
        inputObject.visible = true;
      } else {
        inputObject.visible = false;
      }
    }
  }

  // Other scene update logic ...
}
```

### Pointing rays
Pointing rays should be drawn only for `XRInputSource` objects with the `inputMode` of 'tracked-pointer'.  The ray should start at the origin of the `XRInputSource`'s inherited `XRSpace` and terminate at the point of intersection if one is available.  If no hit test result is found, a default ray length should be used. It is often desirable to display a pointing ray for only the preferred input source and to disable the pointing ray during a drag interaction.

```js
function updatePointingRay(hitTestResult) {
  let pose = frame.getPose(preferredInputSource, xrReferenceFrame);

  // Toggle the pointing ray in the imaginary 3D engine
  if (!pose || preferredInputSource.inputMode != "tracked-pointer" || activeDragInteraction) {
    scene.pointingRay.visible = false;
  } else {
    scene.pointingRay.visible = true;
    scene.pointingRay.setTransform(pose.transform);
    if (hitTestResult) {
      scene.pointingRay.length = MatrixMathLibrary.distance(hitTestResult.transform, hitTestResult.transform);
    } else {
      scene.pointingRay.length = scene.pointingRay.defaultLength;
    }
  }
}
```

## Appendix A: Proposed partial IDL
This is a partial IDL and is considered additive to the core IDL found in the main [explainer](explainer.md).
```webidl
//
// Session
//

partial interface XRSession {
  readonly attribute XRSpace viewerSpace;

  Promise<XRHitTestSource> requestHitTestSource(XRHitTestOptionsInit options);

  FrozenArray<XRInputSource> getInputSources();
  
  attribute EventHandler onselect;
  attribute EventHandler onselectstart;
  attribute EventHandler onselectend;
  attribute EventHandler oninputsourceschange;
};

//
// Frame
//

partial interface XRFrame {
  // Also listed in the spatial-tracking-explainer.md
  XRViewerPose? getViewerPose(optional XRReferenceSpace referenceSpace);
  XRPose? getPose(XRSpace space, XRSpace relativeTo);

  FrozenArray<XRHitTestResult> getHitTestResults(XRHitTestSource hitTestSource, optional XRSpace relativeTo);
  Promise<FrozenArray<XRHitTestResult>> requestAsyncHitTestResults(XRHitTestOptionsInit options, optional XRSpace relativeTo);
};

//
// Input
//

[SecureContext, Exposed=Window,
 Constructor(optional DOMPointInit origin, optional DOMPointInit direction),
 Constructor(XRRigidTransform transform)]
interface XRRay {
  readonly attribute DOMPointReadOnly origin;
  readonly attribute DOMPointReadOnly direction;
  readonly attribute Float32Array matrix;
};

enum XRInputMode {
  "gaze",
  "tracked-pointer",
  "screen"
};

enum XRHandedness {
  "",
  "left",
  "right"
};

[SecureContext, Exposed=Window]
interface XRInputSource : XRSpace {
  readonly attribute XRHandedness handedness;
  readonly attribute XRInputMode inputMode;
  readonly attribute XRSpace? gripSpace;
};

//
// Hit Testing
//

enum XREnvironmentFeatureType {
  "point",
  "vertical-plane",
  "horizontal-plane",
};

dictionary XRHitTestOptionsInit {
  required XRSpace space;
  XRRay offsetRay = new XRRay();
  XREnvironmentFeatureType featureType = "point"; 
};

[SecureContext, Exposed=Window]
interface XRHitTestOptions {
  readonly attribute XRSpace space;
  readonly attributeXRRay offsetRay = new XRRay();
  readonly attribute XREnvironmentFeatureType featureType; 
};

[SecureContext, Exposed=Window]
interface XRHitTestSource {
  readonly attribute XRHitTestOptions hitTestOptions;
};

[SecureContext, Exposed=Window]
interface XRHitTestResult {
  readonly attribute XRHitTestOptions hitTestOptions;
  readonly attribute XRRigidTransform transform;
  readonly attribute FrozenArray<DOMPointReadOnly>? boundsGeometry;
};

//
// Events
//

[SecureContext, Exposed=Window, Constructor(DOMString type, XRSessionEventInit eventInitDict)]
interface XRSessionEvent : Event {
  readonly attribute XRSession session;
};

dictionary XRSessionEventInit : EventInit {
  required XRSession session;
};

[SecureContext, Exposed=Window, Constructor(DOMString type, XRInputSourceChangeEventInit eventInitDict)]
interface XRInputSourceChangeEvent : Event {
  readonly attribute XRSession session;
  readonly attribute FrozenArray<XRInputSource> removed;
  readonly attribute FrozenArray<XRInputSource> added;
  XRHitTestSource getHitTestSource(XRInputSource inputSource);
};

dictionary XRInputSourceChangeEventInit : EventInit {
  required XRSession session;
  required FrozenArray<XRInputSource> removedInputSources;
  required FrozenArray<XRInputSource> addedInputSources;
  required FrozenArray<XRHitTestSource> addedHitTestSource;
};

[SecureContext, Exposed=Window,
 Constructor(DOMString type, XRInputSourceEventInit eventInitDict)]
interface XRInputSourceEvent : Event {
  readonly attribute XRFrame frame;
  readonly attribute XRInputSource inputSource;
};

dictionary XRInputSourceEventInit : EventInit {
  required XRFrame frame;
  required XRInputSource inputSource;
};
```