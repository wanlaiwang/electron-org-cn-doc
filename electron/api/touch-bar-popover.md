## Class: TouchBarPopover

> Create a popover in the touch bar for native macOS applications

Process: [Main]({{site.baseurl}}/docs/tutorial/quick-start#main-process)

### `new TouchBarPopover(options)` _Experimental_

*   `options` Object
    *   `label` String (optional) - Popover button text.
    *   `icon` [NativeImage]({{site.baseurl}}/docs/api/native-image) (optional) - Popover button icon.
    *   `items` [TouchBar]({{site.baseurl}}/docs/api/touch-bar) (optional) - Items to display in the popover.
    *   `showCloseButton` Boolean (optional) - `true` to display a close button on the left of the popover, `false` to not show it. Default is `true`.

### Instance Properties

The following properties are available on instances of `TouchBarPopover`:

#### `touchBarPopover.label`

A `String` representing the popover's current button text. Changing this value immediately updates the popover in the touch bar.

#### `touchBarPopover.icon`

A `NativeImage` representing the popover's current button icon. Changing this value immediately updates the popover in the touch bar.
