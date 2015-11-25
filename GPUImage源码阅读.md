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



