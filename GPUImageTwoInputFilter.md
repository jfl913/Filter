``` objective-c
#import "ViewController.h"
#import <GPUImage.h>

@interface ViewController ()

@property (weak, nonatomic) IBOutlet UIImageView *testImageView;

@property (nonatomic, strong) GPUImageAddBlendFilter *filter;
//@property (nonatomic, strong) GPUImageMonochromeFilter *filter;

@property (nonatomic, strong) GPUImagePicture *sourcePicture;
@property (nonatomic, strong) GPUImagePicture *sourcePicture1;

@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];

    self.filter = [[GPUImageAddBlendFilter alloc] init];

    // 要处理的图片
    self.sourcePicture = [[GPUImagePicture alloc] initWithImage:[UIImage imageNamed:@"jfl"]];
    // 用作滤镜本身的图片
    self.sourcePicture1 = [[GPUImagePicture alloc] initWithImage:[UIImage imageNamed:@"blackboard1024"]];

    [self.sourcePicture addTarget:self.filter];
    [self.sourcePicture1 addTarget:self.filter];
}

- (IBAction)filter:(id)sender
{
    // 这个必不可少
    [self.filter useNextFrameForImageCapture];

    // 两张图片都要调用processImage
    [self.sourcePicture processImage];
    [self.sourcePicture1 processImage];

    UIImage *resultImage = [self.filter imageFromCurrentFramebuffer];
    self.testImageView.image = resultImage;
}

@end
```

