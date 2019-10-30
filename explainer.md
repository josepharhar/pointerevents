# Extension document explainer
This document is a brief explainer for new features added in the [extension document](https://w3c.github.io/pointerevents/extension.html).

## Coalesced Events
Nowadays there are high frequency devices that generate user inputs much faster than the frame refresh rate in the apps.
Hence some browsers chose to not deliver all those events to the web pages as they realized it is going to be a lot of extra work that doesn't necessarily have an effect on the visual output.
For example Chromium delays all move events until right before rAF and Firefox does the same if the first event in the frame causes any changes to the DOM.

### Problem
Since most of the web apps care only about the latest state of the input before generating the next frame, the events happened during the frame are currently coalesced together into one event that gets dispatched to the page.
In this case the dispatched event has the latest screenX/Y coordinates and its movementX/Y is the sum of movementX/Y of all the events that got coalesced into the dispatch event.
This greatly helped reduce the amount of unnecessary work by UA for routing, hit-testing and such for most of the web apps.
However, there is small set of apps like drawing apps that might still care about the historical details of all those events.
For instance they need to know the exact path of the pointer to be able to draw a smooth line.
So although coalescing events and dispatching the coalesced event once per frame greatly helps in terms of performance we still need an API on the dipatched event to expose the coalesced events information in case the app wants to take advantage of that.

### Solution
We propose to provide an API called `getCoalescedEvents` on the `PointerEvent` object that exposes any events that have been coalesced into the dispatched event.
This API also matches other platforms such as [getHistoricalX/Y](https://developer.android.com/reference/android/view/MotionEvent.html#getHistoricalX(int,%20int)) on Android and [similar APIs](https://developer.apple.com/documentation/uikit/touches_presses_and_gestures/handling_touches_in_your_view/getting_high-fidelity_input_with_coalesced_touches) on iOS.

There are two options for exposing the coalesced events:
1. Return a list of dictionary of interesting and useful attributes such as coordinates.
2. Return a list of `PointerEvent` objects representing the actual events that have been coalesced.

Each item in the result list corresponds to one coalesced event.
The first option has the benefit of not having an actual `Event` object and hence there is no need to deal with a set of functionalities that we don't need on coaleaced events, such as propagation functions in the specification.
The second option gives a more usable way to the developers since if their code is capable of handling a `PointerEvent` object already they could easily just pass the coalesced events to that and it would work fine.

Because of the ease of use for developers, and following positive feedback, the working group chose the second option.
You can find examples on how to use the proposed API in the [proposed specification](https://w3c.github.io/pointerevents/extension.html#examples).


## Predicted Events
This is the proposal to match other platforms' predicted events API for the events to reduce the general user perception of latency.

### Problem
Imagine a drawing app that draws the points as it gets them from an input device such as mouse.
There is always some amount of latency compared to when the users draw on a paper due to inherent UA input pipeline latency and graphics pipeline latency and buffering.
Having some sort of the prediction to draw preemptively a little in the future would reduce the perception of the latency. Having the API as part of the web platform enables UA to use some of the underlying platform APIs as well as using other attributes that might not be exposed to the web directly in the prediction algorithm.

### Solution
We propose to provide an API called `getPredictedEvents` on the `PointerEvent` object. This API format will have the same concerns and API alternatives as the `getCoalescedEvent` API introduced earlier.
In order to match it with `getCoalescedEvents` as well as exposing an ergonomic API we will use the same format as `getCoalescedEvents` by returning a list of `PointerEvent` objects.
The events in the result list will have timestamps in the future, and the coordinates will be the position the UA predicts the pointer is heading to. 
This solution will match other platforms such as [prediction](https://developer.apple.com/documentation/uikit/touches_presses_and_gestures/handling_touches_in_your_view/minimizing_latency_with_predicted_touches) on iOS.
You can find examples on how to use the proposed API in the [proposed specification](https://w3c.github.io/pointerevents/extension.html#examples).


## PointerRawUpdate
Nowdays some UAs delay the events until before the next rAF as explained earlier in this document.
For example Chromium delays all move events until right before rAF and Firefox does the same if the first event in the frame causes any changes to the DOM.
However, there is another low latency need that this way of delaying events would break.

### Problem
There are some apps that may need to handle the events regardless of any visual effect.
For example, some of the apps might be sending events over the network like in the cloud gaming use cases and some others might interact with audio devices.
Delaying the delivery of the events would hurt the experience on these apps. These apps need a way of receiving the events without any delay.

### Solution
Although delivering the events at the speed they happen might have a general burden on UAs and running input pipeline for all those events might hurt the performance, this is still necessary for the aformentioned apps.
We propose to have a new event type called `pointerrawupdate` to capture this high frequency event.
Note that this needs to be a new event to guarantee if the web apps chose to listen to it they understand that the UA needs to do extra work for them and may not be able to optimize the input pipeline as much.
This event is supposed to be dispatched as soon as possible without any intentional delaying from the UA for any visual optimization such as delaying event to the next rAF.
 


