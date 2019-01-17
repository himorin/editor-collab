# WebXR Device API - Hit Testing
This document explains the portion of the WebXR APIs that enable developers to hit test against the real-world environment. For context, it may be helpful to have first read about [WebXR Session Establishment](explainer.md), [Spatial Tracking](spatial-tracking-explainer.md), and [Input Mechanisms](input-explainer.md).

## Introduction
"Hit testing" (aka "raycasting") is the process of finding intersections between 3D geometry and a ray, comprised of an origin and direction. Conceptually, hit testing can be done against virtual 3D geometry or real-world 3D geometry. As WebXR does not have any knowledge of the developer's 3D scene graph, it does not provide APIs for virtual hit testing. It does, however, have information about the real-world and provides a method for developers to hit test against it. Most commonly in WebXR, developers will hit test using `XRInputSource`s or the `XRSession.viewerSpace` to track where a cursor should be drawn on hand-held devices, or even to bounce a virtual object off real-world geometry. In WebXR, 'inline' and 'immersive-vr' sessions are limited to performing virtual hit tests, while 'immersive-ar' sessions can perform both virtual and real-world hit tests. 

## Real-world hit testing
A key challenge with enabling real-world hit testing in WebXR is that computing real-world hit test results can be performance-impacting and dependant on secondary threads in many of the underlying implementations.  However from a developer perspective, out-of-date asynchronous hit test results are often, though not always, less than useful. 

WebXR addresses this challenge through the use of the `XRHitTestSource` type which acts somewhat like a subscription mechanism. The presence of an `XRHitTestSource` signals to the user agent that the developer intends to query `XRHitTestResult`s in subsequent `XRFrame`s.  The user agent can then precompute `XRHitTestResult`s based on the `XRHitTestSource` properties such that each `XRFrame` will be bundled with all "subscribed" hit test results. When the last reference to the `XRHitTestResult` has been released, the user agent is free to stop computing `XRHitTestResult`s for future frames.

### Requesting a hit test source
To create an `XRHitTestSource` developers call the `XRSession.requestHitTestSource()` function.  This function accepts an `XRHitTestOptionsInit` dictionary with the following values:
* The `space` is required and is the `XRSpace` to be tracked by the hit test source.
* The `offsetRay` is optional and is the `XRRay` from which the hit test should be performed. This ray is defined relative to the `space`. If an `offsetRay` is not provided, the default is an `XRRay` with a forward direction and an origin coincident with the `space` origin.

In this example, an `XRHitTestSource` is created slightly above the center of the `viewerSpace`. This is because the developer has chosen to leave space for buttons along the bottom of the hand-held AR device screen while still giving the perception of a centered cursor (see [Rendering cursors and highlights](#Rendering-cursors-and-highlights)

```js
let viewerHitTestSource = null;

let hitTestOptions = { space:xrSession.viewerSpace, offsetRay:new XRRay({ origin: {y: 0.5} }) };
xrSession.requestHitTestSource(hitTestOptions).then((hitTestSource) => {
  viewerHitTestSource = hitTestSource;
});
```

### Automatic hit test source creation
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

### Hit test results
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

  xrFrame.requestAsyncHitTestResults(hitTestSources[preferredInputSource], xrReferenceSpace).then((results) => {
    // do something with results
  });

  // Other scene update logic ...
}
```

## Combining virtual and real-world hit testing
A key component to creating realistic presence in XR experiences, relies on the ability to know if a hit test intersects virtual or real-world geometry. For example, developers might want to put a virtual object somewhere in the real-world but only if a different virtual object isn't already present. In future spec revisions, when real-world occlusion is possible with WebXR, developers will likely be able to create virtual buttons that are only "clickable" if there is no physical object in the way. 

There are a handful to different techniques which can be used to determine a combined hit test result. For example, a developer may choose to weight hit test results differently if a user is already interacting with a particular object. In this explainer, a simple example of combining hit test results is provided. Because WebXR does not have any knowledge of the developer's 3D scene graph, this sample uses the `XRFrame.getPose()` function to create a ray and passes it into the 3D engine's virtual hit test function.  If a virtual hit-test is found it is returned, otherwise the sample returns the first real-world hit test result (real-world hit-test results are returned closest to farthest in the array).

```js
function updateScene(timestamp, xrFrame) {
  // Scene update logic ...

  let hitTestResult = getHitCombinedHitTestResult(xrFrame);

  // Other scene update logic ...
}

function getHitCombinedHitTestResult(frame, inputSource, hitTestSource) {
  // Try to get virtual hit test result
  if (inputSource) {
    let inputSourcePose = frame.getPose(inputSource.source, xrReferenceSpace);
    if (inputSourcePose) {
      var virtualHitTestResult = scene.virtualHitTest(new XRRay(inputSource.transform));
      return { result:virtualHitTestResult, virtualTarget:virtualHitTestResult.target }
    }
  }

  // Try to get real-world hit test result
  if (hitTestSource) {
    var realHitTestResults = frame.getHitTestResults(hitTestSource, xrReferenceSpace);
    if (realHitTestResults.length > 0) {
      return { result:realHitTestResults[0] };
    }
  }

  return {};
}
```

## Grab-and-Drag
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
 
## Appendix A: Proposed partial IDL
This is a partial IDL and is considered additive to the core IDL found in the main [explainer](explainer.md).
```webidl
//
// Session
//

partial interface XRSession {
  Promise<XRHitTestSource> requestHitTestSource(XRHitTestOptionsInit options);

  // Also listed in the input-explainer.md
  attribute EventHandler oninputsourceschange;
};

//
// Frame
//

partial interface XRFrame {
  FrozenArray<XRHitTestResult> getHitTestResults(XRHitTestSource hitTestSource, optional XRSpace relativeTo);
  Promise<FrozenArray<XRHitTestResult>> requestAsyncHitTestResults(XRHitTestOptionsInit options, optional XRSpace relativeTo);
};

//
// Hit Testing
//

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
// Feature types
//

enum XREnvironmentFeatureType {
  "point",
  "vertical-plane",
  "horizontal-plane",
};

//
// Events
//

partial interface XRInputSourceChangeEvent {
  XRHitTestSource getHitTestSource(XRInputSource inputSource);
};
```