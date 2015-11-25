GPUImage在一个滤镜里仅仅支持两个纹理。InstaFilters子类化了一些GPUImage的关键组件，扩展它们的能力。

同时，实现了高级的MVC结构。

> But GPUImage is written for general purposes OpenGL ES handling and currently only supports up to 2 textures in one filter while Instagram takes up to 5 textures and requires dynamic filter switching in run time. To port Instagram's recipes downwards into GPUImage's low-level APIs, InstaFilters subclasses a few key componets of GPUImage, extends their abilities and makes sure they are Instagram-friendly.
> 
> At the same time, InstaFilters also implements the upper-level MVC structure with the help of UIKit. And takes care of the background asynchronous tasks of recording, writing, mixing and saving of audio and video files with the help of the AVFoundation framework.

