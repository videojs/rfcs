- Feature Name: Spatial Navigation
- Start Date: 2023-09-12
- PR:

# Summary
[summary]: #summary

This feature aims to improve user experience and accessibility on smartTV devices by enabling seamless navigation through interactive elements within the Video.js player using remote control arrow keys.

# Motivation
[motivation]: #motivation

Smart TV devices commonly use directional input, such as arrow keys on a remote control. By implementing spatial navigation using the spatial navigation polyfill, we aim to ensure that users can intuitively navigate through player controls and elements on smartTV devices.

###Goals
- Enable seamless and intuitive navigation within the Video.js player on smart TV platforms.
- Create a new SpatialNavigation component.
- Implement the spatial navigation polyfill.
- Redesign the Video.js user interface to suit smart TV platforms and facilitate spatial navigation.

###Non-Goals
- Keycode mapping for smartTV devices that use non-standardized keycodes. The spatial navigation polyfill uses 37: 'left', 38: 'up', 39: 'right', and 40: 'down'.

# Guide-level Explanation
[guide-level-explanation]: #guide-level-explanation

Smart TV users will be able to navigate through Video.js player controls using arrow keys on their remote controls. The player will have an initial focus; when a user presses, for example, the "right" arrow key, the focus will shift to the next interactive control, allowing them to easily navigate through player UI elements. Focused UI elements will be selectable using the ENTER/OK button on the RCU.

In order to allow the user to navigate out of the progress bar (Slider), the UP and DOWN buttons will not call the stepForward and stepBack functions, as is now. The LEFT and RIGHT buttons will retain the same functions but only when the progress bar (Slider) is in focus.

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

##Dependency Management and Configuration Options
For this feature, additional dependencies will be added: spatial-navigation-polyfill and flat-polyfill. Flat-polyfill is needed to accommodate devices lacking support for Array.prototype.flat.

To introduce configurational flexibility, a new configuration option `enableRCUNavigation` will be introduced. This option will enable or disable the spatial navigation feature.

##Bundle Size Considerations
At this stage, we haven't precisely determined the impact of the spatial navigation feature on the bundle size. If the integration of spatial navigation results in a significant growth in the bundle size, we‘ll introduce an alternative Video.js bundle with a spatial navigation feature included.

##Code and UI changes

This new feature will follow these steps:
- Introduce a public option named `enableRCUNavigation`.
- When a player is set up with the rcu navigation feature active, initiate the SpatialNavigation instance.
- Collect every focusable component and incorporate them into the SpatialNavigation instance.
- Focus first available navigable candidate.
- Once an arrow is pressed we should go through Basic Spatial Navigation Heuristics (call navigate(direction))
  - Find Candidates
  - Select the best candidate
  - Focus the best candidate

###In terms of changes to the code:

We need to make sure that events can still move upwards when spatial navigation is turned on (`enableRCUNavigation` is set to true). Currently, it’s blocking all keydown events with `event.stopPropagation()`, except for TAB key.
Block the UP and DOWN keys to execute `stepForward()` and `stepBack()` functions, as these keys will be needed for navigation from and to the progress bar (Slider).

###In terms of adding the code

Spatial-navigation interface and Component class extension

```
interface Positions {
boundingClientRect: DOMRectReadOnly;
center: DOMRectReadOnly;
}

interface Component {
// Is considered as focusable component.
getIsFocusable(): boolean;
// Is currently visible/enabled/etc...
getIsAvailableToBeFocused(): boolean;
// on focus handler
handleFocus(): void;
// on blur handler
handleBlur(): void;
// on submit handler
handleEnter(): void;
// component's element boundingClientRect and center
getPositions(): Positions;
// focus component's element
focus(): void;
}

enum SpatialNavigationDirection {
Left = 'left',
Right = 'right',
Up = 'up',
Down = 'down'
}

interface SpatialNavigation {
// current focusable components:
getComponents(): Set<Component>;
// start list keydown events
start(): void;
// stop listen key down events
stop(): void;
// add focusable component
add(component: Component): void;
// remove focusable component
remove(component: Component): void;
// temporary pause spatial navigation
pause(): void;
// resume spatial navigation
resume(): void;
// clear current list of focusable components
clear(): void;
// run spatial navigation Heuristics (re-use https://github.com/WICG/spatial-navigation/blob/main/polyfill/spatial-navigation-polyfill.js#L147)
move(direction: SpatialNavigationDirection): void;
}
```

###In terms of changes the UI
- Since LEFT and RIGHT keys are reserved for video seeking, we’ll have to change the UI and move the progress bar on top of the buttons in order to allow the users to use those keys to navigate between elements when buttons are focused and seek the video when the progress bar is focused
- Current player controls are too small for big screens, so we’re aiming to make those bigger
- Vjs-control-bar is using “display: flex;”, in order to make spatial-navigation-polyfill work properly we have to add a “gap: 1px;”

# Drawbacks
[drawbacks]: #drawbacks

- There might be some performance issues, especially with lower-end devices, due to the polyfill usage.
- There might also be compatibility issues with certain platforms, especially older ones.
- Some older platforms, like Samsung Tizen and LG webOS, don't support Array.prototype.flat, which is why we introduced the flat-polyfill.

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

Despite the above-mentioned drawbacks, introducing spatial navigation using the polyfill seems like the most effective way to provide a standardized behaviour across different smart TV platforms. The alternative to this would be to develop our own custom navigation logic, which might not be as efficient or effective as the polyfill.

The Video.js player with spatial navigation can attract a wider audience and improve the overall user experience for smart TV users.

# Prior art
[prior-art]: #prior-art

At the moment, the Video.js player works on smart TV devices only when it's combined with a custom UI, which handles all keydown events. Introducing spatial navigation directly in the player will make it more versatile and accessible to a larger audience.

# Unresolved questions
[unresolved-questions]: #unresolved-questions

- Performance optimization for spatial navigation on smart TV devices needs thorough evaluation.
- Compatibility testing across different smart TV platforms
