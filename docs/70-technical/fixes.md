---
description: Fixes that VRCFury applies to all avatars by default
slug: /fixes
---

# Automatic Fixes

When VRCFury is present in a project, it applies several fixes to resolve common VRChat and Unity bugs.

## Avatars

**Note**: The fixes in this section **only** apply if your avatar contains any VRCFury component.

* VRChat does not support Animator Override Controllers
  * VRCFury automatically flattens Animator Override Controllers on your descriptor, making them compatible with VRChat.
* Transition times for muscle animations are ignored by unity when placed on layer 0 of an animator
  * VRCFury inserts an empty base layer when this issue is detected
* The VRC avatar descriptor is corrupted and includes two FX layers when you change the avatar rig to generic and back to humanoid
  * VRCFury corrects this issue by keeping the proper FX controller and replacing the other with the missing Action layer
* Assets that were used in an animator previously, but the layer was deleted, are still included in the upload bundle
  * VRCFury corrects this by walking all controllers and building a new copy containing only the layers and states that are actually used
* Controller layers that are empty still take processing time in game
  * VRCFury automatically removes layers that are guaranteed to be empty and have no side-effects
* Animations that target invalid objects still take processing time
  * VRCFury automatically removes animation properties that are guaranteed to be invalid (targeting an object that does not exist)
* States in an additive layer marked as WD off can cause blendshape values to multiply by 3x, and cause unrelated transforms to move continuously
  * VRCFury removes the additive controller that VRChat adds by default (unused by virtually everyone)
  * VRCFury automatically marks all additive states as WD on (only if you opt into Fix Write Defaults)
* States marked as WD off containing a direct blendtree can cause unrelated animations in the controller to break
  * VRCFury automatically marks all direct blendtrees as WD on (only if you opt into Fix Write Defaults)
* The VRCSDK fails with a cryptic error message when menu icons are too large
  * VRCFury detects this condition and shows a useful error message with the icon paths
* VRChat throws away parameters in game if there are more than 256 unique params, even if they are not network synced
  * VRCFury detects this condition and shows a useful error message
* VRChat throws away contacts in game if you have more than 256 total
  * VRCFury detects this condition and shows a useful error message
* VRChat IK and OSC will crash if you have two parameters with the same name, except one has underscores where the other has spaces
  * VRCFury automatically renames parameters conflicting in this way
* VRChat IK and OSC will crash if you have a parameter containing the text `[]`
  * VRCFury automatically renames parameters conflicting in this way
* Controller states and blendtree children with a "None" (missing) animation can break transforms and other animations in avatars using WD off
  * VRCFury automatically replaces missing animations with an empty 1-second clip in these scenarios
* Unity "claims" transforms and muscles for the lowest controller that has them unmasked, even if they are not actively being animated
  * This is a common issue between an avatar's Gesture and FX layers, because when users have no mask in FX, it "claims" all transforms,
    breaking any transforms that are attempted in Gesture.
  * VRCFury solves this issue by moving all non-muscle animations from Gesture into the top of FX, ensuring they continue working as they did previously.
  * Applying a mask to all FX layers (as recommended in other guides) is not a foolproof solution, as masking layers is impossible in some
    scenarios ([read more about it here](/technical/wd#broken-properties-in-masked-layers)).
* Using WD off in a controller state that is placed in a layer at or below a state with WD on causes unity to break all animations in the controller
  * VRCFury detects this scenario and offers the option to solve it automatically by aligning the WD of each setting.
  * Additive layers and direct blendtrees are excluded from this rule, and must always be WD on (as described above).
* MMD worlds do not work properly for avatars using WD off, disable the Gesture controller, and (in some songs) disable FX layer 1 and 2
  * VRCFury automatically resolves all of these issues (only if you opt into VRCFury MMD Compatibility)
* When you delete an int controller parameter and recreate it as a bool, all controller transitions using that parameter will say "True", even though internally they are broken and will never transition.
  * VRCFury automatically corrects these transitions to work if the bool is true, as indicated by the editor.
* When you delete or change the type of a parameter, but have conditions expecting the old type, attempting to use the controller will cause unity to abort the controller entirely, breaking the avatar in the editor and in game
  * VRCFury automatically removes these transitions, as they are never valid
* When you delete a float parameter and recreate it as an int, unity will internally respect the old float thresholds even though the editor displays them as floored
  * VRCFury automatically updates the internal thresholds to reflect the value displayed in the editor
* If one of the controllers on the avatar descriptor is set to "None", unity animation states temporarily reset in between frames, causing Audio Sources to restart every frame
  * VRCFury automatically replaces "None" controllers with an empty replacement
* Non-mobile materials are still included in the quest upload bundle if they are used in animations
  * VRCFury automatically removes all non-mobile materials from quest builds to ensure they are not included in the upload
* VRChat ignores Animator Tracking Control behaviors that run too soon after the avatar is loaded
  * VRCFury buffers all tracking control changes and repeats them several times as the avatar is loaded
* Animator Tracking Control behaviors from various unrelated plugins can break MMD dances
  * VRCFury buffers all tracking control changes and triggers them only once the avatar is not in an MMD station
* Avatars with WD on will incorrectly always select the default value of material properties from the first material slot, even if that material doesn't have the property.
  * Strangely, this issue only happens in-game and is not reproducible in the editor
  * VRCFury resolves this issue by detecting these cases and forcefully records the default state in a top defaults layer, even if WD is on.
* When a physbone with Is Animated unchecked is placed on a humanoid bone (like the hips), it prevents that bone from animating even if the multi-child mode is set to Ignore.
  * VRCFury automatically marks all physbones targeting humanoid bones as animated, to ensure that locomotion and full body animations can operate correctly.
* Because of the way stereo cameras work in VR, objects with small bounding boxes can disappear when still in view, even if the bounding box is technically correct.
  * VRCFury automatically increases the bounds of all renderers by up to one meter.
  * Renderers using lights are left untouched (DPS/TPS/SPS).
  * The bounds are not increased if they will impact the avatar's overall bounds, ensuring the performance rank will not be affected.
* Physbones not marked as Animated targeting humanoid bones completely break full-body animations, even if they are set to Ignore multi-child mode
  * VRCFury automatically marks all physbones targeting humanoid bones as animated
* VRCSDK fails upload if textures do not have "Streaming Mip-Maps" enabled
  * VRCFury automatically clones textures and enables "Streaming Mip-Maps" on them if required by the VRCSDK
* VRCSDK fails upload if menu icons are too large or uncompressed
  * VRCFury automatically clones menu icons and resizes / compresses them to fit VRCSDK limits
* VRCSDK fails upload if audio clips do not have "Load in Background" checked
  * VRCFury automatically audio clips and enables this option when required
* VRCFury automatically upgrades all unity constraints on the avatar to VRC Constraints during the upload
  * In the rare situation that this breaks things, it can be disabled in Tools > VRCFury > Settings.
* Certain particle system settings will cause VRChat to disable your avatar for "Security Checks"
  * VRCFury will automatically correct these settings to not violate the VRChat restrictions.
* VRCAnimatorPlayAudio audio clips are included in the upload during android builds, even though they cannot play
  * VRCFury removes all VRCAnimatorPlayAudio behaviours from android builds to save space in your upload bundle
* Params in the FX controller with invalid parameter types break synchronization of the mirror clone
  * VRCFury automatically removes all controller parameters with invalid types
* VRChat Expression Parameters are synced between platforms by order and type (sorta) rather than by name, making it very difficult to keep platform sync working properly with non-destructive frameworks
  * VRCFury automatically aligns and corrects the order of parameters within mobile builds to match the last desktop build of the same blueprint ID. Missing parameters are replaced with a placeholder, and extra parameters have network sync disabled (with a warning).
* Expression menu items with an invalid type show up as "Button" in the editor, but break in game and in editors
  * VRCFury converts invalid menu item types to Button to match what is shown in the editor

## VRCSDK

* The VRCSDK incorrectly applies android validations while in windows mode if you select the android tab in the inspector of a texture asset
  * VRCFury automatically syncs this setting back to the proper value when the vrcsdk dialog loads and when a build begins
* VRCSDK contacts do not properly distinguish between "Self" and "Others" when testing in editor play mode
  * VRCFury automatically adjusts the playerId on avatar contacts in play mode to make "Self" and "Others" work properly.
* Having the VRCSDK in a project creates a useless folder called XR in the Assets root
  * VRCFury automatically moves the Assets/XR folder into its own temporary package to keep your project clean
* The VRCSDK opens an empty Animator window that cannot be closed if scripts reload while viewing a Parameter Driver
  * VRCFury patches this issue to not occur
* The VRCSDK switches back to the Authentication tab every time you reload scripts
  * VRCFury automatically switches the VRCSDK to the Builder tab if you are already logged in when scripts reload
* The VRCSDK spams the console with "Animator is not playing an AnimatorController" before the animator loads if your avatar contains contacts
  * VRCFury patches this issue to not occur
* The VRCSDK spams the unity console with AmplitudeAPI errors if your network uses pihole or adguard
  * VRCFury hides these messages from the console, since they are non-actionable and clutter actual errors that the user is looking for
* Because of a unity bug, VRCSDK's AnimatorPlayAudioEditor can throw an exception after scripts reload if the editor was previously used on a behaviour that has since been deleted (unity does not clean up old behaviour editors properly)
  * VRCFury patches AnimatorPlayAudioEditor's OnEnable callback to properly return early if the targeted behaviour has been deleted
* Some versions of the VRCSDK [break the testing of contacts in play mode](https://feedback.vrchat.com/sdk-bug-reports/p/race-condition-in-contactmanager-often-crashes-contacts-in-the-editor)
  * VRCFury patches the VRCSDK to resolve this issue
* A bug in the VRCSDK prints an error message related to "stopwatch" when using play mode for the first time after scripts reload
  * VRCFury patches the VRCSDK to fix this bug
* [Dynamics and contacts stop working](https://feedback.vrchat.com/sdk-bug-reports/p/376-dynamics-only-work-the-first-time-you-enter-play-mode) after the first time you enter play mode in VRCSDK 3.7.6
  * VRCFury patches the VRCSDK to fix this bug
* The VRCSDK collider editor automatically reverts custom collider transforms any time you look at the component
  * VRCFury prevents this from happening when in play mode, so custom collider transforms will appear properly while testing
* Avatar colliders can be unset, or set to incorrect transforms, if the rig is modified and then vrcsdk collider section is not viewed afterward
  * VRCFury forces the VRCSDK to recalculate collider transforms at the beginning of each build (they can still be persistently customized by using a VRCFury Global Collider or Advanced Collider component)
* The VRCSDK collider editor doesn't mirror offsets properly if the avatar is not at x=0
  * VRCFury patches the mirroring logic so that colliders mirror properly across the avatar origin. Colliders with mirroring enabled are also recalculated at the start of each avatar build.

## Unity

* VRCFury removes the built-in unity "Toggle", "Dropdown" and "Toggle Group" components from the Add Component menu, as they are not usable
  during avatar creation, and cause confusion when creating an avatar Toggle.
* Unity 2019 does not properly apply changes from nested prefabs when those changes are made externally (for example, using version management) until the outer prefab is reimported
  * VRCFury automatically recursively triggers unity to reimport prefabs in inside-out order prior to loading their vrcfury components
  * This bug is resolved in Unity 2022
* Users accidentally hide console errors and then complain that there are no script errors when checking
  * VRCFury ensures the Error logging level is enabled in the unity console whenever scripts are reloaded
* Users accidentally go into pause mode forever and then complain that Gesture Manager does not work properly
  * VRCFury ensures pause mode and "Error Pause" are both disabled when scripts are reloaded
* Reloading scripts in the editor breaks things if you are in play mode
  * VRCFury automatically changes unity's "Compilation During Play" setting to "Recompile After Finished Playing" if it is set to "Recompile And Continue Playing"
* Unity can segfault during the avatar build if certain methods are called after using the VRCSDK's custom thumbnail camera
  * VRCFury unsets the unity main camera before executing to prevent this issue
* Unity Animator Controller editors spam the console with "Invalid Layer Index" and shows incorrect layer weight and parameter values while viewing an avatar controlled by a controller mixer (Gesture Manager or av3emu)
  * VRCFury patches the animator controller editor to work properly with controller mixers
* A bug in Mono breaks the VRCSDK dialog when Harmony is used, the system locale is non-english, and the project path contains non-english characters
  * VRCFury patches Mono to prevent this issue from occurring to the VRCSDK and other plugins using Assembly.GetName()
* Unity throws a NullReferenceException in "UnityEditor.Graphs.Edge.WakeUp" when reloading scripts if an empty animator has been open recently
  * VRCFury fixes this unity bug and prevents the error
* Unity reloads all scripts and scenes when entering play mode
  * VRCFury automatically disables play mode domain reload and scene reload, which greatly speeds up entering play mode, and is (at this point) compatible with basically all unity plugins
* Unity's "Mesh Data Optimization" takes FOREVER during avatar uploads, and is not a required process
  * Mesh data optimization removes unused UV channels from meshes if it determines the UVs are unused by all attached shaders
  * The duration of this process scales with the number and complexity of shaders used. Since poiyomi creates a unique shader for each material, this complexity grows extremely fast.
  * Unused UV channels are already quite rare (why would a model have them if they are not used?), and an entire UV channel is only half the size of one single blendshape.
  * VRCFury automatically disables this optimizer to massively speed up avatar builds
* Dragging a controller state, by even a single pixel, while previewing a controller in play mode, immediately disables the animator and breaks gesture manager
  * VRCFury prevents you from dragging controller states while previewing them in live-link
* Unity's controller layer list scrolls all the way to the top any time you move or delete a layer
  * VRCFury patches this issue

## AudioLink

* AudioLink in the editor does not automatically attach to newly-created renderers after entering play mode.
  * VRCFury reinitializes all AudioLink components in the scene after an avatar's preprocessor hooks have run.

## Av3Emulator

* Av3Emualator throws errors and breaks if an animator in the scene disappears or has its controller changes after play mode has begun
  * VRCFury reinitializes Av3Emu after an avatar's preprocessor hooks have run

## Package Importer
* Importing Poiyomi or the VRCSDK from an avatar package when they are already installed breaks the project for users who do not know to exclude them from the import
  * VRCFury prevents Poiyomi or VRCSDK asset files from being imported if they are already detected in the project.

## Poiyomi

* Poiyomi locks down all materials when preprocessor hooks are run in play mode in versions before 9
  * VRCFury patches Poiyomi's LockMaterialsOnUpload to never apply while in play mode.
