#### [GPUImage 自定义滤镜](http://www.itiger.me/?p=143)

GPUImage的自定义滤镜需要使用到OpenGL着色语言(GLSL)来编写Fragment Shader(片段着色器), 且后缀为.fsh.

> 也可以写到滤镜本身的类文件里

``` objective-c
GPUImageFilter *customFilter = [[GPUImageFilter alloc] initWithFragmentShaderFromFile:@"GPUImageCustomFilter"];
_filteredImage = [customFilter imageByFilteringImage:_originImage];
```

GLSL脚本文件(GPUImageCustomFilter.fsh)如下:

``` 
varying highp vec2 textureCoordinate;
uniform sampler2D inputImageTexture;

void main()
{
    lowp vec4 textureColor = texture2D(inputImageTexture, textureCoordinate);
    lowp vec4 outputColor;
    outputColor.r = (textureColor.r * 0.393) + (textureColor.g * 0.769) + (textureColor.b * 0.189);
    outputColor.g = (textureColor.r * 0.349) + (textureColor.g * 0.686) + (textureColor.b * 0.168);
    outputColor.b = (textureColor.r * 0.872) + (textureColor.g * 0.534) + (textureColor.b * 0.131);
    outputColor.a = 1.0;

    gl_FragColor = outputColor;
}
```

对于一个在GPUImage framework里的滤镜来说，前两行是必须的。

`textureCoordinate varying` : for the current coordinate within the texture , normalized to 1.0

`inputImageTexture uniform` : for the actual input image frame texture

剩下的代码抓取传进来的texture的这个位置的像素的颜色，用这种方式来操作它，产生褐色的色调，并把它写入用在下一阶段的处理。

一个需要注意的地方，当你添加片段着色器到Xcode工程里，Xcode会认为它们是源代码文件。你应该手动把它们从Compile Sources build phase移动到Copy Bundle Resources ，使它们包含在application bundle里。

`varying highp vec2 textureCoordinate`是顶点（Vertex）着色器和片段着色器之间传递数据。处理后的图片的坐标。

`uniform sampler2D inputImageTexture`声明是外部程序传递过来给shader的变量，只能用，不能改。inputImageTexture包含图像的全部信息。需要被处理的图片。

`texture2D(inputImageTexture, textureCoordinate)`纹理取样器

`lowp vec4 textureColor = texture2D(inputImageTexture, textureCoordinate);`

`textureColor.rgb`原图的RGB值

`gl_FragColor`是片段着色器定义的变量，是该片段最终的颜色值。图片处理后textureCoordinate的颜色。

不要在滤镜里加注释，而且中文可能会出问题

---

初始化方法有以下几种：

``` objective-c
/**
 Initialize with vertex and fragment shaders

 You make take advantage of the SHADER_STRING macro to write your shaders in-line.
 @param vertexShaderString Source code of the vertex shader to use
 @param fragmentShaderString Source code of the fragment shader to use
 */
- (id)initWithVertexShaderFromString:(NSString *)vertexShaderString fragmentShaderFromString:(NSString *)fragmentShaderString;

/**
 Initialize with a fragment shader

 You may take advantage of the SHADER_STRING macro to write your shader in-line.
 @param fragmentShaderString Source code of fragment shader to use
 */
- (id)initWithFragmentShaderFromString:(NSString *)fragmentShaderString;
/**
 Initialize with a fragment shader
 @param fragmentShaderFilename Filename of fragment shader to load
 */
- (id)initWithFragmentShaderFromFile:(NSString *)fragmentShaderFilename;
```

---

相对Core Image，GPUImage允许自定义滤镜并提供简单的接口。但它不支持脸部检测。

图片或者视频帧从source objects上传，source objects是`GPUImageOutput`的子类。

`GPUImageOutput`包含：

>  `GPUImageVideoCamera` : live video from camera
> 
> `GPUImageStillCamera` : take photos with camera 
> 
> `GPUImagePicture` : still images
> 
> `GPUImageMovie` : movie

source objects 上传still image frames作为textures，然后把这些textures交给在处理链的下一些对象。

Filters和在处理链中的other subsequent elements 遵守`<GPUImageInput>`协议。

`<GPUImageInput>`协议让遵守的对象采用前一个链提供或处理过的texture，并对它进行处理。



For example, an application that takes in live video from the camera, converts that video to a sepia tone, then displays the video onscreen would set up a chain looking something like the following:

`GPUImageVideoCamera -> GPUImageSepiaFilter -> GPUImageView`



For blending filters and others that take in more than one image, you can create multiple outputs and add a single filter as a target for both of these outputs. The order with which the outputs are added as targets will affect the order in which the input images are blended or otherwise processed.



``` objective-c
GPUImageVideoCamera *videoCamera = [[GPUImageVideoCamera alloc] initWithSessionPreset:AVCaptureSessionPreset640x480 cameraPosition:AVCaptureDevicePositionBack];
videoCamera.outputImageOrientation = UIInterfaceOrientationPortrait;

GPUImageFilter *customFilter = [[GPUImageFilter alloc] initWithFragmentShaderFromFile:@"CustomShader"];
GPUImageView *filteredVideoView = [[GPUImageView alloc] initWithFrame:CGRectMake(0.0, 0.0, viewWidth, viewHeight)];

// Add the view somewhere so it's visible

[videoCamera addTarget:customFilter];
[customFilter addTarget:filteredVideoView];

[videoCamera startCameraCapture];
```

Once you want to capture a photo, you use a callback block like the following:

``` objective-c
[stillCamera capturePhotoProcessedUpToFilter:filter withCompletionHandler:^(UIImage *processedImage, NSError *error){
    NSData *dataForJPEGFile = UIImageJPEGRepresentation(processedImage, 0.8);

    NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
    NSString *documentsDirectory = [paths objectAtIndex:0];

    NSError *error2 = nil;
    if (![dataForJPEGFile writeToFile:[documentsDirectory stringByAppendingPathComponent:@"FilteredPhoto.jpg"] options:NSAtomicWrite error:&error2])
    {
        return;
    }
}];
```

> GPUImage framework在旧的设备上不能处理宽度或者高度大于2048像素的图片。

#### Processing a still image

The first way you can do this is by creating a still image source object and manually creating a filter chain:

``` objective-c
UIImage *inputImage = [UIImage imageNamed:@"Lambeau.jpg"];

GPUImagePicture *stillImageSource = [[GPUImagePicture alloc] initWithImage:inputImage];
GPUImageSepiaFilter *stillImageFilter = [[GPUImageSepiaFilter alloc] init];

[stillImageSource addTarget:stillImageFilter];
[stillImageFilter useNextFrameForImageCapture];
[stillImageSource processImage];

UIImage *currentFilteredVideoFrame = [stillImageFilter imageFromCurrentFramebuffer];
```

Note that for a manual capture of an image from a filter, you need to set `useNextFrameForImageCapture` in order to tell the filter that you will be needing to capture from it later. By default, GPUImage reuses frame buffers within filters to conserve memory, so if you need to hold on to a filter's frame buffer for manual image capture, you need to let it know ahead of time.

``` objective-c
- (void)processImageUpToFilter:(GPUImageOutput<GPUImageInput> *)finalFilterInChain withCompletionHandler:(void (^)(UIImage *processedImage))block;
{
    [finalFilterInChain useNextFrameForImageCapture];
    [self processImageWithCompletionHandler:^{
        UIImage *imageFromFilter = [finalFilterInChain imageFromCurrentFramebuffer];
        block(imageFromFilter);
    }];
}
```

这个方法不需要设置｀useNextFrameForImageCapture｀，我们可以看到它的源码里已经调用了这个方法。

For single filters that you wish to apply to an image, you can simply do the following:

``` objective-c
GPUImageSepiaFilter *stillImageFilter2 = [[GPUImageSepiaFilter alloc] init];
UIImage *quickFilteredImage = [stillImageFilter2 imageByFilteringImage:inputImage];
```

#### 写一个自定义滤镜

一个自定义的滤镜用以下方法来初始化：

``` objective-c
GPUImageFilter *customFilter = [[GPUImageFilter alloc] initWithFragmentShaderFromFile:@"CustomShader"];
```

文件以`.fsh`结尾。

除此之外，你还可以用以`initWithFragmentShaderFromString`来提供片段着色器作为一个字符串。

#### 跟OpenGL ES交互

GPUImage可以通过GPUImageTextureOutput和GPUImageTextureInput类来导出和导入texture。

---

``` objective-c
- (UIImage *)processUsingGPUImage:(UIImage*)input {
 
  // 1. Create the GPUImagePictures
  GPUImagePicture * inputGPUImage = [[GPUImagePicture alloc] initWithImage:input];
 
  UIImage * ghostImage = [self createPaddedGhostImageWithSize:input.size];
  GPUImagePicture * ghostGPUImage = [[GPUImagePicture alloc] initWithImage:ghostImage];
 
  // 2. Set up the filter chain
  GPUImageAlphaBlendFilter * alphaBlendFilter = [[GPUImageAlphaBlendFilter alloc] init];
  alphaBlendFilter.mix = 0.5;
 
  [inputGPUImage addTarget:alphaBlendFilter atTextureLocation:0];
  [ghostGPUImage addTarget:alphaBlendFilter atTextureLocation:1];
 
  GPUImageGrayscaleFilter * grayscaleFilter = [[GPUImageGrayscaleFilter alloc] init];
 
  [alphaBlendFilter addTarget:grayscaleFilter];
 
  // 3. Process & grab output image
  [grayscaleFilter useNextFrameForImageCapture];
  [inputGPUImage processImage];
  [ghostGPUImage processImage];
 
  UIImage * output = [grayscaleFilter imageFromCurrentFramebuffer];
 
  return output;
}
```

GPUImageAlphaBlendFilter takes two inputs, in this case the top image and bottom image, so that the texture locations matter. -addTarget:atTextureLocation: sets the textures to the correct inputs.![GPUFilterChain-283x320](/Users/jfl/Downloads/GPUFilterChain-283x320.png)