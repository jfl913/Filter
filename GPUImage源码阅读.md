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

