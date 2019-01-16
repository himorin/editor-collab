# RENAME ME TO HIT TESTING EXPLAINER
# WebXR Device API - Environment awareness
This document explains the portion of the WebXR APIs that give developers access to environmental awareness. For context, it may be helpful to have first read about [WebXR session establishment](explainer.md), [spatial tracking](spatial-tracking-explainer.md), and [input mechanisms](input-explainer.md).

## Hit Testing
"Hit testing" (aka "raycasting") is the process of finding intersections between 3D geometry and a ray, comprised of an origin and direction. Hit testing can be done against virtual 3D geometry or real-world 3D geometry. Most commonly in WebXR, developers will hit test from `XRInputSource`s or from the `XRSession.viewerSpace`.  Some purposes of hit testing might be to determine user intention, track where a cursor should be drawn on hand-held devices, or even to bounce a virtual object off real-world geometry. In WebXR, 'inline' and 'immersive-vr' sessions are limited to performing virtual hit tests, while 'immersive-ar' sessions can perform both virtual and real-world hit tests. 

### Virtual hit testing
A virtual hit test is a hit test that returns results from intersections with virtual 3D geometry. WebXR does not have any knowledge of the developer's 3D scene graph, but does have information about the real-world location of `XRSpace` objects.  Using the `XRFrame.getPose()` function, developers can create a ray and pass it into their 3D engine's virtual hit test function. Developers should take care to check the result from `getPose()` as it may return `null` in cases where tracking has been lost or the `XRSpace` cannot be found.

This example shows a virtual hit test being performed based on a `preferredInputSource` previously identified by the 3D engine.

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

  xrFrame.requestAsyncHitTestResults(hitTestSources[preferredInputSource], xrReferenceSpace).then((results) => {
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