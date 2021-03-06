GPUImagePicture

`glBindTexture(GL_TEXTURE_2D, outputTexture);`

GPUImageOutput

``` objective-c
- (void)addTarget:(id<GPUImageInput>)newTarget;
{
    cachedMaximumOutputSize = CGSizeZero;
    NSInteger nextAvailableTextureIndex = [newTarget nextAvailableTextureIndex];
    [newTarget setInputTexture:outputTexture atIndex:nextAvailableTextureIndex];
    [targets addObject:newTarget];
    [targetTextureIndices addObject:[NSNumber numberWithInteger:nextAvailableTextureIndex]];
}	
```

多图渲染要复写这个方法。

``` objective-c
- (void)setInputTexture:(GLuint)newInputTexture atIndex:(NSInteger)textureIndex;
{
    if (textureIndex == 0)
    {
        filterSourceTexture = newInputTexture;
    }
    else if (filterSourceTexture2 == 0)
    {
        filterSourceTexture2 = newInputTexture;
    } 
    else if (filterSourceTexture3 == 0) {
        filterSourceTexture3 = newInputTexture;
    }
    else if (filterSourceTexture4 == 0) {
        filterSourceTexture4 = newInputTexture;
    }
    else if (filterSourceTexture5 == 0) {
        filterSourceTexture5 = newInputTexture;
    }
    else if (filterSourceTexture6 == 0) {
        filterSourceTexture6 = newInputTexture;
    }
}
```

A `uniform` is a global GLSL variable declared with the "uniform" storage qualifier. These act as parameters that the user of a shader program can pass to that program. They are stored in a `program` object.

Uniforms are so named because they do not change from one execution of a shader program to the next within a particular rendering call. This makes them unlike shader stage inputs and outputs, which are often different for each invocation of a program stage.



OpenGL做通用计算的步骤主要如下：

读取数据->顶点着色器->片段着色器->渲染到纹理->从纹理读写数据。

（2）创建纹理对象以及设置参数

这里需要创建两个纹理对象，一个用于输入图像，另外一个用于保存输出的结果，即将保存结果的纹理对象绑定到帧缓存对象上，即实现渲染到纹理的目标。

最后是将输出纹理绑定到帧缓存对象上并激活输入纹理对象。