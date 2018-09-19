title: OC-常用的Foundation类
date: 2016/2/29 15:49:06           
categories: Objective-C
---

# OC-常用的Foundation类 #

- 可变类与不可变类
>- Foundation类都是容器，他们具有两种类型：不可变类和可变类
>- 许多可变类与不可变类都是成对的
>- 常见的有：NSString-NSMutableString, NSArray-NSMutableArray

- 类簇
>- NSString，NSArray，NSDictory，NSSet, NSNumber, NSData这些类都实现为类簇
>- 类簇是一种吧复杂性隐藏在一个简单接口后面的方式。
>- 类簇的私有子类都实现了由公共可见类定义的接口，但是他们的内部实现可能有所不同
>- 当我们请求公共类的一个实例时，将接收到类簇的私有子类之一的实例
>- 至于我们到底使用的是哪个子类，这一般是由init方法确定
>- 类簇的注意事项
>    - 不应该在实现为簇的类的实例上使用NSObject内省方法isMemberOfClass
>    - 子类化一个实现为类簇的类是非常复杂的。

- NSString
>- NSString有两种类型：字面量NSString和普通的NSString对象。
>- 创建字面量NSString： NSString* greeting = @"Hello";
>     - 字面量NSString实例是由编译器在只读内存中创建的静态对象
>     - 字面量字符串将在程序的生存期内持续存在，不受OC的内存管理的影响
>     - 给字面量字符串发送一条retain或release消息时没有问题的。
>- 至于普通的NSString对象，创建的方式有很多种

>- 使用示例

	//NSLog
	NSLog( @"The sky is %@.", @"blue");	
	NT: 1. NSLog的格式字符串本身也是NSString
		2. NSLog给对应于“%@”说明符的参数发送一条description消息。
		   这样，由description方法返回的NNString的文本将替换掉“%@”
	//常用方法
	NSString* string = @"Hello Objective-C";
	[string length];
	[string uppercaseString];  //转大写
	[string1 stringByAppendingString string2]; //将string2追加到string1上
	//与C字符串的转换
	char* cString = "Helloc Universe";
	NSString* nsString  =[NSString stringWithUTF8String: cString]; //C->OC
	cString = [nsString UTF8String]; //OC->C

- NSMutableString
>- 类似与java中的String， NSString是不可变的。
>- NSString的子类NSMutableString是可变的，（即这个对象中保存的文本是可以动态的变化的）

	NSMutableString* mutableString = [NSMutableString stringWithString: @"Shun the Bandersnatch."];
	[mutableString insertString: @"frumious" atIndex: 9];

- NSArray
>- 普通的C数组没有关于数组长度的运行时信息，也不会进行边界检查。
>- NSArray就有许多强大的特性
>    - NSArray只能保存OC对象
>    - NSArray对象会自动进行边界检查， 超界就报错
>    - NSArray对象是密集的，
>    - 对于放在NSArray中的对象，NSArray将保留这些对象的所有权(retain)
>    - arrayWithObjects: 的列表通过nil来终止
    
	NSArray* fruitBasket = [NSArray arrayWithObjects: @"Apple", @"Orage", @"Banana", nil];
	NSUInteger numFruits = [fruitBasket count];

- NSMutableArray
>- 可以动态的向这个数组中添加对象 
>- 但是：只可以在现有的索引范围内插入对象或者把他们添加到数组的末尾

- NSDictionary与NSMutableDictionary
>- 类似Map，用来处理键-值对的集合
>- 特点：当把一个记录添加到字典中时，字典将创建键的副本。即如果使用对象作为键，对象的类就必须实现copyWithZone: 方法

	NSMutableDictionay* favoriteFruits = [NSMutableDictionay dictionaryWithCapacity: 3];
	[favoriteFruits setObject: @"Orage" forKey: @"FavoriteCitrusFruit"];
	[favoriteFruits setObject: @"Apple" forKey: @"FavoritePieFruit"];
	
	[favoriteFruits setObject: @"Apple2" forKey: @"FavoritePieFruit"]; //替换一个对象
	
	id pieFruit = [favoriteFruits objectForKey: @"FavoritePieFruit"];  //获取对象

- NSSet与NSMutableSet
>- 集合的特性
>    - 集合成员是无序的，无法从集合中检索特定对象
>    - 集合中的对象是不能重复的
>    - 将对象添加到集合中，对象将被保留

	NSMutableSet* favoriteAnimals = [NSMutableSet setWithCapacity: 3];
	[favoriteAnimals addObject: @"dog"];
	[favoriteAnimals addObject: @"dog"];  //无效的操作
	[favoriteAnimals addObject: @"cat"];

- NSNumber
>- 用来封装基本C类型的对象
>- 实例是不可变的。如果需要包装一个不同的数值，就必须创建一个新的NSNumber

	NSNumber* floatObject = [NSNumber numberWithFloat: 9.9f];
	float y = [floatObject floatValue];

- NSNull
>- 他用来解决集合中对象无状态的问题
>- 不能直接把nil放在数组， 字典或集合中
>- 因此他就出现了
>- 这个类只有一个类方法:null

	[myArray addObject: nil]; //编译时不会报错， 但运行时会直接挂掉
	[myArray addObject: [NSNull null]]; //集合中将会包含 "<null>"

- NSData与NSMutableData
>- 用来包装字节块的OC对象

	char* someBytes = (char*)malloc(100);
	//将someBytes复制到由NSData对象创建并拥有的缓冲区中。NSData对象被取消分配时，将释放该缓冲区
	//someBytes最终还是得由你来释放
	NSdata* data = [NSData dataWithBytes: someBytes length: 100]; 
	
	//直接使用someBytes所表示的空间作为缓冲区，将NSData对象取消分配时，someBytes被释放
	NSdata* data = [NSData dataWithBytes: someBytes length: 100 freeWhenDone: YES]; 

	//从缓存区中获取空间
	const char* constantBytes = [data bytes];   //空间只读
	char* mutableBytes = [mutableData mutableBytes];

>- 在文件与NSData之间转移内容
>    - 我们可以把字节移入或移出文件

	NSdata* myData = [NSData dataWithContentsOfFile: pathToFile];  //把文件内容移入myData的缓冲区中
	[myData writeToFile: pathToOutputFile atomically: YES]; //YES表示，当操作成功后，将文件的名称重命名为路径参数指定的名称

- NSURL
>- NSURL可用于文件URL和网络URL

	NSURL* someFile = [NSURL fileURLWithPath: @"/Users/rclair/notes.txt"];
	NSURL* website = [NSURL URLWithString: @"http://www.chromaticbytes.com"];

- OC字面量和对象下标
>- 这两个新特性用于简便我们的书写
>- NSArray字面量
> 	- 字面量只能用来创建普通数组，并不能用来创建可变数组
> 	- 字面量方括号之间的对象列表不是利用nil终止的
> 	- 像便利构造函数一样，字面量不会把所有权传递给调用代码，即数组中没有保留字面量

	NSArray* array = @[@"Ralph", @"Norton", @"Alice"];   //非常方便的构造数组

>- NSDictionary
>    - 字面量创建的是不可变字典
>    - 其他特性类似上面的NSArray字面量
	
	NSDictionary* drawingColors = @{ @"fillColor" : [UIColor redColor], @"outlineColor" : [UIColor blackColor]}；

>- NSNumber字面量
>- 更方便了

	NSNumber* count = @76;
	NSNumber* boolYes = @YES;

- 装箱表达式  (有个卵用？)
>- 装箱一个表达式将对表达式求值，并把结果放在一个合适的对象中
>- 从Xcode4.4起， 数值类型（包括BOOL和枚举）和C字符串就可以装箱， 分别产生NSNUmber对象和NSString
>- 语法： @(expression);
>- 可惜的是，没有用于取消装箱NSNumber对象的特殊语法

	NSNumber* count = @(25); //与 	NSNumber* count = @25;相同
	NSString* str = @("Hello World");  //并没有什么意义
	NSString* str = @( functionThatReturnsCString() );  //这就很有用了

- 对象和下标
>- 数组下标

	NSArray* fruitBasket = [NSArray arrayWithObjects: @"Apple", @"Orage", @"Banana", nil];
	id item = fruitBasket[2];  //等价于 [fruitBasket objectAtIndex: 2]
	fruitBasket[2] = @"hehe";   //等价于 [fruitBasket insertObject: @"Joseph" atIndex: 2];

>- 字典下标
>    - 就是以键作为下标
>    - 这个到挺方便

	NSMutableDictionay* favoriteFruits = [NSMutableDictionay dictionaryWithCapacity: 3];
	[favoriteFruits setObject: @"Orage" forKey: @"FavoriteCitrusFruit"];
	[favoriteFruits setObject: @"Apple" forKey: @"FavoritePieFruit"];
	NSString* str = favoriteFruits[@"FavoriteCitrusFruit"];

>- 给自己的类添加下标
	
	id object = array[index];  -> id object = [array objectAtIndexSubscript: index];
	array[index] = newObject;  -> [array setObject: newObject arIndexedSubscript: index]; 
	//因此，只需我们的类实现上面这两个方法即可