title: IOS-UIScrollView 与 UICllectionVew
date: 2016/3/9 8:48:22        
categories: IOS
---

# IOS-UIScrollView 与 UICllectionVew #


## UIScrollView ##
>类似与android中的ScrollView，UIScrollView是一个能够滚动的视图控件，
>可以用来展示大量的内容，并且可以通过滚动查看所有的内容

- 如何使用
>- 设置UIScrollView的contentSize属性，告诉UIScrollView所有内容的尺寸，
>  也就是告诉它滚动的范围（能滚多远，滚到哪里是尽头）
>- 默认情况下，UIScrollView是不能滚动的
>- 但是在使用时你可能会发现你的UIScrollView并不能滚动，可能是由下面原因引起的
>    - 没有设置contentSize
>    - scrollEnabled = NO
>    - 没有接收到触摸事件:userInteractionEnabled = NO
>    - 没有取消autolayout功能（要想scrollView滚动，必须取消autolayout）】
>    - ....

>- UIScrollView的常见属性
>     - @property(nonatomic) CGPoint contentOffset; 
>       这个属性用来表示UIScrollView滚动的位置
>     - @property(nonatomic) CGSize contentSize; 
>       这个属性用来表示UIScrollView内容的尺寸，滚动范围（能滚多远）
>     - @property(nonatomic) UIEdgeInsets contentInset; 
>       这个属性能够在UIScrollView的4周增加额外的滚动区域，跟UIScrollView的回弹效果有关

## UICllectionVew ##

- 简介
>- 类似与Android中的GridView，主要用来显示宫格型数据的
>- 对于每个cell的显示位置，UICllectionVew会自动调节
>- UICllectionVew使用原理类似与UITableView
>    - 需要设置数据源
>    - 可对cell进行分组
>    - IOS提供了UICllectionViewController

- 如何使用
>- 布局样式(参数)
>    - 和UITableView不同的是，UICollectionView在初始化时需要一个布局参数
>    - 并且其内显示的cell的样式也要在初始化时设置


	//这个方法是MyCollectionViewController的初始化方法， MyCollectionViewController继承自UICollectionViewController
	- (id)init
	{
	    // 1.流水布局， 流动填补缺的位置
	    UICollectionViewFlowLayout *layout = [[UICollectionViewFlowLayout alloc] init];

		//对于每个cell的大小，也是由布局参数决定

	    // 2.每个cell的尺寸
	    layout.itemSize = CGSizeMake(80, 80);
	    // 3.设置cell之间的水平间距
	    layout.minimumInteritemSpacing = 0;
	    // 4.设置cell之间的垂直间距
	    layout.minimumLineSpacing = 10;
	    // 5.设置四周的内边距
	    layout.sectionInset = UIEdgeInsetsMake(layout.minimumLineSpacing, 0, 0, 0);
		//还可设置滚动方向等等。。。。。

	    return [super initWithCollectionViewLayout:layout];   
	}

>- cell的创建
>    - 在collectionView初始化完毕后，我们应向它注册一个cell
>    - 注册cell时分为两种情况
>        - 直接alloc,init
>        - 从xib中创建
>    - 创建cell时需要提供一个可从用标志

	//在CollectionViewController中创建cell
	-(void)viewDidLoad
	{
	    [super viewDidLoad];
	    
	    // 注册cell(cell对象来做xib)
	    UINib *nib = [UINib nibWithNibName:@"MJProductCell" bundle:nil];
	    [self.collectionView registerNib:nib forCellWithReuseIdentifier:MJProductCellID];
	    
		//cell对象手动创建
	  //[self.collectionView registerClass:[UICollectionViewCell class] forCellWithReuseIdentifier:MJProductCellID];
	    
	}

>- 数据源相关方法
>    - 在访问指定位置的cell时，和TableView有些不同
>        - 应这访问， 组： indexPath.section； 组中的第几个： indexPath.item

	- (NSInteger)numberOfSectionsInCollectionView:(UICollectionView *)collectionView
	{
	    return 1;  //不重写这个方法，就是返回1
	}

	- (NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section
	{
	    //返回 numberOfItemsInSection ,即一组中cell的个数
	}
	
	- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath
	{
	    // 1.获得cell， 直接取出在初始化方法中已经创建的cell
	    MJProductCell *cell = [collectionView dequeueReusableCellWithReuseIdentifier:MJProductCellID forIndexPath:indexPath];
	    
	    // 2.向cell传递模型
	   //TODO
	    
	    return cell;
	}


 







