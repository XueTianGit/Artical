title: IOS-对View进行单元格排列（九宫格）
date: 2016/3/9 8:49:27            
categories: IOS
---

# IOS-对View进行单元格排列（九宫格） #
	
	@implementation MJViewController
	
	- (void)viewDidLoad
	{
	    [super viewDidLoad];
	    
	    // 添加应用信息
	    
	    // 0.总列数(一行最多3列)
	    int totalColumns = 3;
	    
	    // 1.应用的尺寸
	    CGFloat appW = 85;
	    CGFloat appH = 90;
	    
	    // 2.间隙 = (控制器view的宽度 - 3 * 应用宽度) / 4
	    CGFloat marginX = (self.view.frame.size.width - totalColumns * appW) / (totalColumns + 1);
	    CGFloat marginY = 15;
	    
	    // 3.根据应用个数创建对应的框框(index 0 ~ 11)
	    for (int index = 0; index<self.apps.count; index++) {
	        // 3.1.创建view
	        MJAppView *appView = [MJAppView appViewWithApp:self.apps[index]];
	        
	        // 3.2.添加view
	        [self.view addSubview:appView];
	        
	        // 3.3.设置frame
	        int row = index / totalColumns;
	        int col = index % totalColumns;
	        // 计算x和y
	        CGFloat appX = marginX + col * (appW + marginX);
	        CGFloat appY = 30 + row * (appH + marginY);
	        appView.frame = CGRectMake(appX, appY, appW, appH);
	        
	        // 3.4.设置数据
	//        appView.app = self.apps[index];
	    }
	}
	
	@end