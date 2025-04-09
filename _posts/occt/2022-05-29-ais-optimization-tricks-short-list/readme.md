---
layout: post
date: 2022-05-29
title: AIS optimization tricks – short list
categories: occt
tags: opencascade opensource performance
permalink: /occt/2022-05-29-ais-optimization-tricks-short-list/
thumbnail:
author: Kirill Gavrilov Tartynskih
---

This post is just a collection of *AIS* performance tricks without in-depth clarification. The list could be updated in the future.

<!--break-->

1. Custom AIS objects are the key.
   - [Standard AIS presentations](../2021-11-08-application-interactive-services-or-ais/#standard-objects) coming with OCCT give a good start,
     but learning how to create own objects and use existing presentation builders is essential to bring diversity and achieve optimal performance in complex applications.
2. `AIS_InteractiveObject`'s are expensive.
   - Combine small presentations into a single object to reduce memory consumption and improve rendering performance.
   - One `AIS_InteractiveObject` doesn't mean a single selectable entity – you may define selection of object parts.
   - Putting too much geometry into a single `AIS_InteractiveObject` isn't good either – consider splitting too large objects into smaller pieces.
3. Combine small primitive arrays sharing common aspects into larger one.
   - If you have hundreds and thousands of `Graphic3d_Group::AddPrimitiveArray()` calls in your object, then you are doing something wrong.
   - Each `Graphic3d_Group::AddPrimitiveArray()` maps into [individual draw calls](../2021-10-26-tkopengl-frame-render-graph/#drawing-a-primitive-array).
     These calls are expensive – the impact on rendering performance would be similar to creating numerous `AIS_InteractiveObject`'s per array (but with smaller memory overhead).
   - Drawing a single primitive array is normally faster than multiple ones.
   - E.g. create a single `Graphic3d_ArrayOfTriangles` from multiple `TopoDS_Face`'s in a shape,
     instead of one-to-one mapping between faces and arrays; the same applies to other arrays.
4. Avoid recomputing an entire *AIS object* to change presentation aspects.
   - If you want to change colors of individual sub-parts of an *AIS object* – put them into individual `Graphiv3d_Group` and assign unique `Graphic3d_Aspects`.
   - Create these aspects at object construction time (NOT within `::Compute()` method).
     This would allow applying changes in individual aspects dynamically just by calling `AIS_InteractiveObject::SynchronizeAspects()`.
5. Take into account operation context.
   - Applications usually don't allow changing everything at once.
     Make smart choices of a strategy optimal for specific context.
   - E.g., grouping small presentations into a larger *AIS object* is good for rendering performance of static geometry, but sub-optimal for partial geometry changes.
   - Don't be afraid to switch strategies on-the-go.
     When operation does small incremental changes of geometry – split large objects into subparts.
     When operation is completed or switched to another object – combine small parts back.
6. Partial changes of large mesh objects.
   - Splitting of large mesh into smaller *AIS objects* might be a reasonable optimization.
   - `Graphic3d_ArrayOfPrimitives` are usually created on the fly within the `::Compute()` method,
     uploaded onto GPU memory for optimal rendering performance and then released from CPU memory.
     However, an application might consider keeping these arrays as object class fields as optimization.
   - `Graphic3d_Buffer::InvalidatedRange()` could be used for partial updates of a primitive array
     onto GPU memory after modifications – without recomputing the entire presentation object.
7. [Optimal approach for highlighting](../2021-11-16-ais-object-computing-selection/#alternative-approaches-to-highlighting) selected / detected parts.
   - Highlighting of the entire *AIS object* reuses existing presentation and just alters color / material properties.
     Pretty fast, but doesn't allow customization or selection of parts.
   - Computing a dedicated presentation within `SelectMgr_EntityOwner::HilightWithColor()` allows maximum customization, but requires extra memory and time to build a presentation on-the-fly.
     Optimal for implementing dynamic highlighting.
   - Modifying aspects of the main presentation within `SelectMgr_EntityOwner::HilightWithColor()` requires extra implementation efforts,
     but brings better performance and memory usage.
   - In complex scenarios, all three approaches listed above could be combined to achieve best performance with maximum customization level.
8. Disable `Prs3d_Drawer::IsAutoTriangulation()`.
   - Auto-triangulation helps to see shaded `TopoDS_Shape` straight-ahead, but becomes a source of hidden performance and visualization issues.
   - Application would better take care of triangulation and manage quality necessary to specific geometry / operation.
   - Multi-threaded option of `BRepMesh_IncrementalMesh` would show much better numbers when called for a group of shapes instead of individual shapes,
     and would handle boundaries of shared shapes appropriately.
   - `BRep_Tool::Triangulations()` could be used to store triangulations of different quality within the shape.
9. Managing Z-Layers.
   - CAD applications often have to display heavy geometry and complex models, which the computer is unable to render at a good framerate.
   - *OCCT 3D Viewer* has a mechanism of *Immediate Layers* for drawing mutable presentations on top of a static scene
     (`V3d_View::RedrawImmediate()`) without redrawing the entire scene (`V3d_View::Redraw()`).
   - *AIS* by default uses this mechanism for dynamic highlighting of detected objects when moving mouse cursor – maintaining a good interactivity even for too heavy scenes.
   - Applications might use the mechanism of immediate layers to maintain updates of mutable presentations
     (e.g. dragging / modifying a small object within a specific operation) by displaying objects in `Graphic3d_ZLayerId_Top` / `Graphic3d_ZLayerId_Topmost` layers.
   - Building BVH trees for frustum culling might take some time.
     Mark dynamically modified objects with `AIS_InteractiveObject::IsMutable()` flag;
     split a large number of objects into several Z-Layers to reduce time spent on BVH tree rebuilds.
     Don't forget to mark an object back as non-mutable after dynamic operation is completed.
10. Defer viewer updates and recomputing *AIS objects*.
   - Immediately calling `AIS_InteractiveContext::Redisplay()` after each small modification might be tempting,
     but may lead to poor performance when multiple modifications are done before a new viewer frame will be actually displayed on the screen.
   - Consider using `AIS_InteractiveObject::SetToUpdate()` property to invalidate an object's presentation
     and call redisplay them before actually redrawing the viewer.
   - And never redraw the viewer (`V3d_View::Redraw()`) after modification of an individual AIS object – redraw content only after all objects have been updated.
   - `AIS_ViewController` might help organizing rendering loop.
11. Defer building of BVH trees in Selection.
   - BVH trees used for selection considerably improve performance of this operation, but building trees might take some time.
   - AIS implements [multi-level BVH trees structure](https://dev.opencascade.org/doc/overview/html/occt_user_guides__visualization.html#occt_visu_2_2) in Selection to avoid recomputing everything after local changes.
   - AIS defers computing small BVH trees until picking comes close to the object – so that scene could be displayed faster.
     In some cases, this might lead to noticeable delays in picking, when the mouse comes close to a large number of small objects.
   - `SelectMgr_ViewerSelector::ToPrebuildBVH()` could be used to compute deferred BVH trees in background thread.
   - Selection trees for large meshes could be computed in multiple threads by splitting them into several groups in the `Select3D_SensitivePrimitiveArray` constructor.
