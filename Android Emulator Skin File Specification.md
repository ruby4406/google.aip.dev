Android Emulator Skin File Specification:
=========================================
Revisions:
----------
    Revision 2. Dated 2009-12-07
    Dynamic Layout support added 10/2012
    Dynamic Layout support removed 3/2016
    Revision 3. Dated 2016-03-10
Introduction:
-------------
The Android emulator program is capable of displaying a window containing
an image of a fake handset and associated controls (e.g. D-Pad, trackball,
keyboard).
The content of this window is dictated by a "skin", i.e. a collection of
images and configuration data that indicates how to populate the window.
This document specifies how to generate a new skin for the emulator.
General File Format:
--------------------
Each "skin" has a unique name and corresponds to a directory filled with
various files. All skins are located under a parent "skin-dir" directory.
You can use the '-skindir <path>' and '-skin <name>' options when starting
the emulator to specify them. This will instruct the program to look into
    <path>/<name>/
For skin-specific files. Without these options, the emulator will look for
skins in the SDK in a way described later in this document.
The most important file in a skin must be named 'layout', as in:
   <path>/<name>/layout
The format of this file must follow the "aconfig" specification, see
docs/ANDROID-CONFIG-FILES.TXT for details about it.
Parts:
----------------
Each skin file must define a list of 'parts'.
A 'skin part' correspond to a named item that can contain a set of
visual/control elements that can be of the following types:
 - 'background': A background image in PNG format.
 - 'display': An emulated LCD screen area.
 - 'buttons': A set of clickable control areas (e.g. for a D-Pad, or
              a Keyboard)
Each part can be independently positioned and/or rotated in the layout.
A skin layout is simply an arrangement of parts.
Skin Layout:
-------------
The layout is a named sub-key of the top-level 'layouts' key in the
config file, for example:
    layouts {
        main {
            ....
        }
    }
NOTE: previously, you could specify multiple layouts in the "layouts" section.
You can still do that, but all the layouts except the first one will be ignored.
Instead, there will be 3 additional layouts generated automatically from the first one:
rotated by 90, 180, and 270 degrees clockwise.
The layout can have the following keys (and corresponding values):
- 'width': The width of the emulator window in pixels, as an integer
- 'height': The height of the emulator window in pixels, as an integer
- 'color' : Background color to be used to fill the emulator window.
            this is a 32-bit ARGB value, the 0x prefix can be used to
            use hexadecimal notation.
- 'event' : An optional specific Linux event code that is generated whenever
            the emulator switches/initializes this layout. This is used to
            emulate the 'keyboard-lid open/close' events when emulating
            certain devices with a hardware keyboard.
            The value must be of the format:
                 <type>:<code>:<value>
            Where the event type, code and value are numerical values or,
            in certain cases string aliases for Linux input-subsystem event
            codes. You can use the following emulator console commands to
            print valid types and codes:
                event types         -> prints all valid types
                event codes <type>  -> prints all valid codes for <type>
            The typical event to be used is EV_SW:0:1 for portrait mode
            and EV_SW:0:0 for landscape ones. They corresponds to "keyboard
            closed" and "keyboard opened" respectively, and would match a
            device like the T-Mobile G1 or the Verizon Droid.
- 'onion' : Specifies an image to overlay on the display. Each such key may
            contain the following sub-keys:
            - 'image' [required]: Filename of image to overlay
            - 'alpha' [optional]: How opaque the image should be, from 0 (not
              visible at all) to 100 (fully opaque). Default is 50.
            - 'rotation' [optional]: Integer in the 0..3 range, specifying the
              clockwise rotation of the image in 90-degree increments.
- 'part<n>': Individual part references for the layout. They are named
             in incremental numerical order, starting from 'part1', as in
             'part1', 'part2', 'part3', etc...
             Each such key must contain the following sub-keys:
               - 'name': The name of the corresponding part to be displayed
                         as defined in the rest of the configuration file
               - 'x':  Horizontal offset where the part is displayed
               - 'y':  Vertical offset where the part is displayed
               - 'rotation': An optional sub-key which value is a integer
                             in the 0..3 range specifying the clockwise
                             rotation (in 90-degrees increment) to apply to
                             the part before display.
- 'dpad-rotation':
             An option integer in the 0..3 range indicating which
             counterclockwise rotation (in 90-degrees increments) to apply to
             the D-Pad keys for proper usage.
             This is needed because the Android framework considers that
             the DPad is in landscape mode when the device is in landscape
             mode and will-auto-rotate the D-Pad value. This setting is used
             to counter-effect this correction for certain skins which
             do not rotate the DPad in landscape mode.
Skin Parts:
-----------
Each skin part is a sub-key of the top-level 'parts' key in the configuration
file. For example:
    parts {
        foo {
            ...
        }
        bar {
            ...
        }
        zoo {
            ...
        }
    }
Defines three parts named 'foo', 'bar' and 'zoo'.
Each part can have one or more elements of the following type/key name that
will determine its visual appearance:
- 'background':
        A background image in PNG format. This is a tree key that can
        have the following sub-keys:
            - 'image':  Name of the PNG image in the skin directory
            - 'x'    :  Optional horizontal offset in pixels (integer)
            - 'y'    :  Optional vertical offset in pixels (integer)
- 'display':
        An optional rectangular area that will appear on top of the
        background image to display an emulated LCD screen. Valid sub-keys
        are:
            - 'x'       : Optional horizontal offset in pixels (integer)
            - 'y'       : Optional vertical offset in pixels (integer)
            - 'width'   : Width in pixels (integer)
            - 'height'  : Height in pixels (integer)
            - 'rotation': Optional rotation value (0..3) in 90 degrees
                          increments.
- 'buttons':
        Used to define a list of rectangular clickable control areas with
        an optional high-lighting image. Each sub-key must have a unique
        name, and may contain the following sub-sub-keys:
            - 'x'       : Horizontal offset in pixels (integer)
            - 'y'       : Vertical offset in pixels (integer)
            - 'image'   : PNG image of the high-lighting for the button
        Each highlight image will be drawn on top of the background are for
        the button. A typical one has 50% opacity. The highlight will be drawn
        twice to simulate 'clicked' state.
        The image's dimensions are used to determine the size of the control
        area.
        The name of each button must correspond to the list of key symbols
        defined in the _keyinfo_table array defined in android/skin/file.c.
Other top-level keys:
---------------------
A few other top-level keys are supported for legacy reasons, but the
corresponding definition is best defined in the hardware properties/config
file instead:
- 'keyboard.charmap':
    Optional, must be the name of the charmap being used for this
    skin. Currently unused so ignore this.
- 'network.speed':
    Default network speed for this skin. Values correspond to the
    -netspeed <speed> emulator command-line option.
- 'network.delay':
    Default network latency for this skin. Values correspond to the
    -netdelay <delay> emulator command-line option.