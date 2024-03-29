---
Availability: Public
Crumbs: 
Title: Finding and Removing Fully Occluded Meshes
Description: Describes how you can increase rendering performance by removing and simplifying geometry that is fully occluded by other objects in the Level.
Type: 
Version: 4.21
Parent: Enterprise/Datasmith/HowTo
Order: 
Tags: how to
Tags: Datasmith
Tags: intermediate
---

![](Images/jacketing-banner.png)

One way to increase rendering performance in any real-time 3D application is to simply reduce the number of objects that need to be drawn each frame. Usually, the camera doesn't see every object in the 3D scene at the same time. Any objects that are occluded — that is, blocked from the camera's current view by other objects — can safely be skipped during rendering to improve performance without changing the final image.

The Unreal Engine has several built-in ways to remove occluded meshes every frame, such as culling meshes that are outside the camera's view frustum, or meshes that are too far from the camera. However, there are cases where the Unreal Engine can't efficiently determine at runtime which meshes are occluded by other meshes — notably, when one mesh lies inside the bounding box of another mesh. This is a common occurrence with computer-aided design (CAD) data that is brought into Unreal Engine for rendering, where assemblies often contain a variety of small parts that are completely hidden inside casings. If these parts will never be visible in the real-time rendering, you can often improve your rendering performance by hiding them, or removing them from the Level entirely. If you import a fully modeled car into the Unreal Engine for rendering, and don't offer a way for the player or viewer to look under the hood, there is no reason to spend resources every frame rendering the internal parts of the engine.

For example, the engine assembly below contains 542 separate Static Mesh Actors. However, 321 of them are entirely enclosed within the casing, and will never be visible to the camera. Removing the occluded geometry from the Level makes the remaining geometry render much more quickly without changing its visual appearance.

[OBJECT:ComparisonSlider]
 [PARAM:before]
 ![Complete engine, 542 Actors](Images/jacketing-engine-default.png) 
 [/PARAM]
 [PARAM:after]
 ![321 fully occluded Actors](Images/jacketing-engine-occluded.png) 
 [/PARAM]
[/OBJECT]

For cases like these, the Unreal Editor offers an on-demand process that scans a selection of Static Mesh Actors in the Level to determine which ones are fully occluded — that is, which ones cannot be seen from any outside viewpoint. Once the process has identified these fully occluded Actors, you can isolate them on their own Layer, remove them from the Level entirely, or simplify their geometry to remove internal details.

This process is sometimes referred to as *jacketing*.

## Gaps

Often, an outer shell of geometry that hides internal meshes from view is not completely closed. The outer geometry may contain small gaps or discontinuities, but still block a viewer from making out internal details. For example, in this motor, the chain passes through the exterior shell through small holes:

![Gaps in the occluding meshes](Images/jacketing-gaps.png "Gaps in the occluding meshes")

In cases like these, you still want to hide the internal meshes. Therefore, when determining which triangles are occluded, the jacketing algorithm can bridge small gaps, acting as if those gaps were covered by a mesh. This allows you to get the benefit of hiding the internal occluded parts, even if the occluding meshes are not completely sealed.

You can configure the maximum size of the gaps that you want to be ignored during the occlusion testing.

[REGION:note]
You may be tempted to set the gap size threshold to a very large value, just to be safe. However, this threshold is also used in the Mesh target mode (see below) to evaluate which triangles are safe to remove from the Static Meshes. If you set the gap threshold too high, the internal triangles of the geometry may not be simplified as much as possible. Set the gap threshold as close as you can to the actual size of the gaps in your occluding meshes.
[/REGION]

## Targets

You can apply the results of the jacketing operation to either of two targets: the [Static Mesh Actors in the Level](#theleveltarget), or [the geometry in the Static Mesh Assets](#themeshtarget).

### The Level Target

When you run the jacketing tool with the Level target, it conducts occlusion tests on a selected set of Static Mesh Actors in the Level. It analyzes the geometry of those Actors from all angles, to determine which of them are completely hidden from view from all angles. Once it has a list of those occluded Actors, you can choose what to do with them.

In the Unreal Editor UI, you can:

*   Tag the occluded Actors with a new Component Tag, **Jacketing Hidden**.  
    ![Jacketing Hidden tag](Images/jacketing-tag.png "Jacketing Hidden tag")  
    **
*   Put the occluded Actors on a new Layer named **Jacketing**.
*   Hide the occluded Actors from view by turning off their **Actor Hidden in Game** setting.
*   Remove the occluded Actors from the Level.

If you run the tool in Level target mode from a Blueprint or Python script, it simply returns the list of occluded Actors, so your script can determine the appropriate action to take.

The Level target mode is a good choice if you have many little parts that are each represented by an individual Static Mesh Actor, within boxes or shells that have relatively simple geometry.

[REGION:note]
In the Level target mode, the jacketing tool never modifies any Static Mesh Assets. It only runs the occlusion test and determines the fully occluded Actors.
[/REGION]

### The Mesh Target

When you use the jacketing tool in Mesh target mode, it considers occlusion at the level of individual triangles. After conducting its occlusion tests, it removes all triangles that it considers occluded from their individual Static Mesh Assets. This effectively reduces the occluding meshes to empty shells, removing detail from their interior surfaces. 

This is a good option when your casings or occluding meshes have complex interior surfaces, or where you have multiple Actors whose geometry overlaps. Any geometry within the areas of overlap is simplified as much as possible.

The jacketing tool uses a conservative approach to identify triangles that it can safely remove, to avoid the possibility of degrading the visual results. Any triangles that may possibly be visible are left intact. The jacketing tool does not re-triangulate or simplify any geometry. It only removes unnecessary triangles. 

For example, the closed assembly shown below has some complex geometry inside it that will never be seen from outside. By running the Jacketing tool with the Mesh target, you can remove all of the internal detail. Note that even the inward-facing surfaces of the shell have been removed, leaving a single-sided geometry facing outward.

[OBJECT:ComparisonSlider]
 [PARAM:before]
 ![Assembly with complex internal geometry](Images/jacketing-mesh-before.png) 
 [/PARAM]
 [PARAM:after]
 ![After jacketing](Images/jacketing-mesh-after.png) 
 [/PARAM]
[/OBJECT]

You can see the results of the Jacketing tool in the **Output Log** panel, including the number of triangles the tool was able to remove:

[REGION:lightbox]
[![Jacketing results](Images/jacketing-results.png "Jacketing results")](Images/jacketing-results.png)

*Click for full image.*
[/REGION]
[REGION:warning]
In Mesh target mode, the jacketing tool modifies your Static Mesh Assets. If those Assets are used elsewhere in your Level, or in other Levels in your Project, those instances will also automatically be updated to show the new geometry.
[/REGION]

## Jacketing in the Level Viewport

To apply jacketing in the Level Viewport:

1.  Select the Static Mesh Actors in the Level that you want to be considered in the occlusion testing. You'll need to select the meshes that make up the outside enclosure, as well as any meshes that lie inside.
2.  Right-click any of the selected Actors in the Level Viewport or the **World Outliner**, and select **Jacketing**.  
    ![Jacketing in the contextual menu](jacketing-right-click.png "Jacketing in the contextual menu")
3.  In the **Remove occluded meshes** window, configure the sensitivity of the occlusion tests and set the target you want to affect.  
    ![Jacketing settings](Images/jacketing-settings.png "Jacketing settings")  
    
| **Setting** | **Description** |
| --- | --- |
| **Voxel precision** | [INCLUDE:#excerpt_0] |
| **Gap max diameter** | [INCLUDE:#excerpt_1] |
| **Action Level** | Determines whether the tool will use the **Level** target or the **Mesh** target. |
| **Action Type** | If you choose to affect the Level target, also use the **Action Type** drop-down list to determine what should be done with the set of Actors that the jacketing tool determines to be fully occluded. See [The Level Target](#theleveltarget) above for details. |
    
<!--
[EXCERPT:excerpt_1]
Sets the maximum size of the gaps in the occluding volumes that the occlusion tests will consider to be filled.
[REGION:note]
Do not set this value too low. See the [Gaps](#gaps) section above for details.
[/REGION]
[/EXCERPT:excerpt_1]
-->
<!--
[EXCERPT:excerpt_0]
Controls the sensitivity of the occlusion tests. Reduce the value for smaller models and to achieve greater precision.
[REGION:note]
This setting directly affects the time and memory requirements for the collision tests. Start with a relatively large value, and lower the value until you achieve the fidelity you need.
[/REGION]
[/EXCERPT:excerpt_0]
-->
    
4.  Click **Proceed** to launch the occlusion tests.  
    ![Proceed](Images/jacketing-proceed.png "Proceed")
5.  If you selected the Mesh target, your modified meshes will be marked as dirty. Save them before closing the Unreal Editor if you want to keep your changes.

## Jacketing in Editor Scripts

You can carry out the same jacketing operation offered by the Level Viewport (and the World Outliner) in Blueprints and in Python.

[REGION:note]
**Prerequisite:** If you haven't already done so, you'll need to install the **Editor Scripting Utilities Plugin**. For details, see [Scripting and Automating the Editor](Engine/Editor/ScriptingandAutomation).
[/REGION]

Choose your language.

### Blueprints

To use these nodes, your Blueprint class must be derived from an Editor-only class, such as the **PlacedEditorUtilityBase** class. For details, see [Scripting the Editor using Blueprints](Engine/Editor/ScriptingandAutomation/Blueprints).

The main Blueprint node that you'll need to use is **Mesh Processing > Mesh Actor > Simplify Assembly**.

![Simplify Assembly node](jacketing-simplify-assembly-bp.png "Simplify Assembly node")

You need to feed this node with two inputs:

*   An Array that contains all the Actors in the current Level that you want to be considered during the occlusion testing.
*   A **JacketingOptions** object that sets up parameters for the occlusion testing. To set up one of these objects:
    1.  Add a new variable to the Blueprint by clicking the **\+ Variable** button in the **My Blueprint** panel.  
        ![Add variable](jacketing-add-variable.png "Add variable")
    2.  Set the type of the variable to be a reference to a **Mesh Defeaturing Parameter Object**.  
        ![Jacketing Options object reference](Images/jacketing-object-reference.png "Jacketing Options object reference")
    3.  Hold **Control** and drag the variable into the Blueprint graph to create a new node that gets the variable value.  
        ![Drag and drop the variable](Images/jacketing-drag-drop.png "Drag and drop the variable")
    4.  Drag right from the output port of the new variable node, and select from the **Variables** list the **Set** nodes for the settings that you need to modify.  
        ![Drag right for the Jacketing Options API](Images/jacketing-options-api.png "Drag right for the Jacketing Options API")

If you set the **JacketingOptions** to use Level target mode, the **Apply Jacketing on Mesh Actors** node returns an Array of all the Static Mesh Actors that are occluded from all points of view. You can then iterate over this list to do something with the Actors.

For example, the following Blueprint graph collects all the Static Mesh Actors in the Level, runs the jacketing occlusion test with the LEVEL target, and selects those Actors in the Viewport and World Outliner.

[REGION:lightbox]
[![Jacketing example](Images/jacketing-example.png "Jacketing example")](jacketing-example.png)

*Click for full image.*
[/REGION]

### Python 

You can run the occlusion test and jacketing process on any set of Static Mesh Actors in the current Level by calling the `unreal.MeshProcessingLibrary.apply_jacketing_on_mesh_actors()` function. You'll need to pass this function two parameters:

*   An array that contains all the Actors in the current Level that you want to be considered during the occlusion testing.
*   An `unreal.JacketingOptions` object that sets up parameters for the occlusion testing. Create a new instance of this object by calling `unreal.JacketingOptions()`, then set up the properties you want to adjust.

If you set up the `unreal.JacketingOptions.target` property to be `unreal.JacketingTarget.LEVEL`, the function returns an array of all the meshes that it considers fully occluded. You can process this list to do whatever you want with them.

    # Get a list of Actors in the Level -- in this case, actors that
    # have been selected by the user in the Viewport.
    actors = unreal.EditorLevelLibrary.get_selected_level_actors()

    # Create a new object to hold the jacketing options.
    options = unreal.JacketingOptions()

    # Sets the resolution of the voxel grid in centimeters.
    options.accuracy = 0.2

    # Sets the maximum gap that can be considered filled, in centimeters.
    options.merge_distance = 3

    # The target to act on: unreal.JacketingTarget.LEVEL, or unreal.JacketingTarget.MESH
    options.target = unreal.JacketingTarget.LEVEL

    # Performs the jacketing operation.
    # Acting on the LEVEL target makes the function return the list of occluded Actors.
    occluded = unreal.MeshProcessingLibrary.apply_jacketing_on_mesh_actors(actors, options)

	# Do something with the occluded Actors.
    # For example, this loop simply deletes them each occluded Actor from the Level.
    for a in occluded:
    a.destroy_actor()


If you set up the `unreal.JacketingOptions.target` property to be `unreal.JacketingTarget.MESH`, the function does not return a value. It simply removes any triangles that it determines are not visible from the outside.

For example:

    # Get a list of Actors in the Level -- in this case, actors that
    # have been selected by the user in the Viewport.
    actors = unreal.EditorLevelLibrary.get_selected_level_actors()

    # Create a new object to hold the jacketing options.
    options = unreal.JacketingOptions()

    # Sets the resolution of the voxel grid in centimeters.
    options.accuracy = 0.2

    # Sets the maximum gap that can be considered filled, in centimeters.
    options.merge_distance = 3

    # The target to act on: unreal.JacketingTarget.LEVEL, or unreal.JacketingTarget.MESH
    options.target = unreal.JacketingTarget.MESH

    # Performs the jacketing operation.
    # Acting on the MESH target makes the function apply changes directly to the geometry of the
    # Static Mesh Assets that are visible from the outside.
    unreal.MeshProcessingLibrary.apply_jacketing_on_mesh_actors(actors, options)
