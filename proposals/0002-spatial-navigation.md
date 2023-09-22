- Feature Name: Spatial Navigation
- Start Date: 2023-09-12
- PR: https://github.com/videojs/rfcs/pull/2

# Summary
[summary]: #summary

This feature aims to improve user experience and accessibility on smartTV devices by enabling seamless navigation through interactive elements within the Video.js player using remote control arrow keys.

# Motivation
[motivation]: #motivation

Smart TV devices commonly use directional input, such as arrow keys on a remote control. By implementing spatial navigation, we aim to ensure that users can intuitively navigate through player controls and elements on smartTV devices.

### Goals
- Enable seamless and intuitive navigation within the Video.js player on smart TV platforms.
- Create a new SpatialNavigation component.

### Non-Goals
- Keycode mapping for smartTV devices that use non-standardized keycodes.

# Guide-level Explanation
[guide-level-explanation]: #guide-level-explanation

Smart TV users will be able to navigate through Video.js player controls using arrow keys on their remote controls. The player will have an initial focus; when a user presses, for example, the "right" arrow key, the focus will shift to the next interactive control, allowing them to easily navigate through player UI elements. Focused UI elements will be selectable using the ENTER/OK button on the RCU.

Currently, when the Slider is focused, all four directional arrow keys are mapped to the seek functions. To offer more granular control and enhance navigation, a new configuration option named `seek` will be introduced. This option will accept two potential values: `horizontal` or `vertical`.

- When `seek` is set to `horizontal`, only the LEFT and RIGHT arrow keys will invoke the stepForward and stepBack functions, permitting users to progress or regress the playback timeline. On the other hand, the UP and DOWN arrow keys will facilitate navigation out of the Slider, transitioning focus to other UI elements.

- If `seek` is defined as `vertical`, the roles of the arrow keys will be inverted. The UP and DOWN arrow keys will invoke the stepForward and stepBack functions, while the LEFT and RIGHT keys will be reserved for spatial navigation.

This enhancement ensures users not only have precise control over playback but also a seamless navigation experience within the Video.js player interface, allowing users to have different player UI.

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

## Dependency Management and Configuration Options
For this feature, additional dependencies will be needed but won't be included by default: spatial-navigation-polyfill and flat-polyfill. Flat-polyfill is needed to accommodate devices lacking support for Array.prototype.flat.

To introduce configurational flexibility, a new configuration option named `spatialNavigation` will be introduced. This option will be an object containing two properties: 
- `enabled`, which can be set to `true` or `false` to enable or disable the spatial navigation feature, 
- `seek`, which can be set to either `horizontal` or `vertical`.

```
spatialNavigation: {
    enabled: true/false,
    seek: 'horizontal'/'vertical'
}
```

## Bundle Size Considerations
At this stage, we haven't precisely determined the impact of the spatial navigation feature on the bundle size. If the integration of spatial navigation results in a significant growth in the bundle size, we‘ll introduce an alternative Video.js bundle with a spatial navigation feature included.

## Code

This new feature will follow these steps:
- Introduce a public option named `spatialNavigation`.
  - This option will be an object containing two properties `enabled`, which can be set to `true` or `false`, and `seek`, which can be set to either `horizontal` or `vertical`.
- When a player is set up with the rcu navigation feature active, initiate the SpatialNavigation instance.
- Collect every focusable component and incorporate them into the SpatialNavigation instance.
- Focus first available navigable candidate.
- Once an arrow is pressed we should go through Basic Spatial Navigation Heuristics (call navigate(direction) - spatial-navigation-polyfill)
  - Find Candidates
  - Select the best candidate
  - Focus the best candidate

### In terms of changes to the code:

We need to make sure that events can still move upwards when spatial navigation is turned on (`spatialNavigation.enabled` is set to true). Currently, it’s blocking all keydown events with `event.stopPropagation()`, except for TAB key.
Block the UP and DOWN keys to execute `stepForward()` and `stepBack()` functions, when `spatialNavigation.seek` is set to `horizontal` as these keys will be needed for navigation from and to the progress bar (Slider) and vice versa.

### In terms of adding the code

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

# Drawbacks
[drawbacks]: #drawbacks

- By not including the necessary polyfills directly within the feature, users will need to ensure they have the correct polyfills in place, adding an external dependency to the setup process. This can potentially lead to integration issues, especially if there's a mismatch in polyfill versions or if a certain polyfill becomes deprecated.
- Relying on external polyfills could introduce unforeseen performance implications if these polyfills are not optimized for all target platforms.

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
