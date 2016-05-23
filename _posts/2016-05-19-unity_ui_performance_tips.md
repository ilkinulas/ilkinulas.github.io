---
layout: post
title: Unity UI Performance Tips
categories: development unity
---

The new UI System is part of the Unity engine since release [4.6](http://blogs.unity3d.com/2014/11/26/4-6-is-released-with-source-for-ui-system/), and it is [open source](https://bitbucket.org/Unity-Technologies/ui). The new UI System is becoming the defacto standard for building game UIs. It is getting better and better with each Unity release but you still need to pay attention to details while creating UIs unless  you want to end up with an FPS suffering game. This post will list some of the best practices learned through experience while developing UI systems for our games.

### 1. Use texture atlases

Using [Texture Atlases](https://en.wikipedia.org/wiki/Texture_atlas) reduces the number of draw calls. Every object that share the same material are on the same draw call, and can be sent to the GPU at once. UI textures aren't packed into a texture atlas by default, you should do it yourself. [TexturePacker](https://www.codeandweb.com/texturepacker) is a really good tool for createing texture atlases. Text also is a major cause of extra draw calls as they are on a separate atlas which causes the extra batch. 

### 2. Disable UI elements if they are behind and not visible.

Any enabled GUI element causes a draw call, regardless of its position. UI objects that are off-screen and enabled increases the number of draw calls. Possible solutions:

 * Disable game objects that are off-screen. Be careful, executing **SetActive(true/false)** for large number of gameobjects can be slow.
 * Change parent of off-screen gameobjects to a non-UI gameobject. UI camera will no longer detect off-screen objects.

### 3. Use [Frame Debugger](http://docs.unity3d.com/Manual/FrameDebugger.html) to view individual draw calls

Frame Debugger window (Menu : Window -> Frame Debugger) shows how the final frame is rendered, step by step. You can see a list of draw calls made during that frame. With Frame Debugger you can see which objects are causing so many draw calls. Below image is a screen capture of Frame Debugger running for our game's Settings screen. By looking at the rendering order of objects we can optimize the number of draw calls.

![Frame Debugger](/assets/unity_ui_performance/settings-frame-debugger.gif){: .center-image }

### 4. Do not set alpha to 0 to hide ui elements.

If you change the color of an image and set its alpha to 0, it will still cause a draw call. To hide UI elements use **SetActive(false)**.

### 5. Hierarchy does affect the the draw calls.

If you put an image between two text widgets in the canvas hierarchy (even with no overlap) two text widgets will draw seperately. Also a text widget between two images (in the same atlas) results in 3 draw calls:

* Draw call 1 : image 1
* Draw call 2 : text
* Draw call 3 : image 2

