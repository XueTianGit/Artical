title: IOS-数据的持久化
date: 2016/3/9 8:51:22               
categories: IOS
---

# IOS-数据的持久化 #

>- IOS中对数据进行持久化的方式有许多种
>    - XML属性列表（plist）归档
>    - Preference(偏好设置)
>    - NSKeyedArchiver归档(NSCoding)
>    - SQLite3 
>    - Core Data


- 应用沙盒(应用的根目录)
>- 数据持久化肯定要存到一个地方，那存在哪里呢？沙盒中
>- IOS中的应用沙盒，就类似与Android中的/data/data/packagename 目录
>- 每个iOS应用都有自己的应用沙盒(应用沙盒就是文件系统目录)，与其他文件系统隔离。应用必须待在自己的沙盒里，其他应用不能访问该沙盒
>- 应用沙盒的文件系统目录，如下图所示（假设应用的名称叫Layer）

![](http://7xrbxa.com1.z0.glb.clouddn.com/IOS%E6%B2%99%E7%9B%92%E7%9A%84%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F%E7%9B%AE%E5%BD%95.png)

>- 模拟器应用沙盒的根路径在: (apple是用户名, 6.0是模拟器版本)
>    - /Users/apple/Library/Application Support/iPhone Simulator/6.0/Applications
>- 沙盒中各个目录结构的用途如下
>    - 应用程序包：(上图中的Layer)包含了所有的资源文件和可执行文件
>    - Documents：保存应用运行时生成的需要持久化的数据，iTunes同步设备时会备份该目录。例如，游戏应用可将游戏存档保存在该目录
>    - tmp：保存应用运行时所需的临时数据，使用完毕后再将相应的文件从该目录删除。应用没有运行时，系统也可能会清除该目录下的文件。iTunes同步设备时不会备份该目录
>    - Library/Caches：保存应用运行时生成的需要持久化的数据，iTunes同步设备时不会备份该目录。一般存储体积大、不需要备份的非重要数据
>    - Library/Preference：保存应用的所有偏好设置，iOS的Settings(设置)应用会在该目录中查找应用的设置信息。iTunes同步设备时会备份该目录
>- 我们一般在进行数据持久化时都写到Documents目录下

	NSString *home = NSHomeDirectory();   //沙盒根目录（应用的根目录）
	NSString *tmp = NSTemporaryDirectory();  //tmp目录

	//Documents目录的2种获取方式
	// 这种方式不建议采用，因为新版本的操作系统可能会修改目录名
	NSString *documents = [home stringByAppendingPathComponent:@"Documents"]; //利用沙盒根目录拼接”Documents”字符串

	//利用NSSearchPathForDirectoriesInDomains函数
	// NSUserDomainMask 代表从用户文件夹下找
	// YES 代表展开路径中的波浪字符“~”
	NSArray *array =  NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, NO);
	// 在iOS中，只有一个目录跟传入的参数匹配，所以这个集合里面只有一个元素
	NSString *documents = [array objectAtIndex:0];

	//Library/Caches：(跟Documents类似的2种方法)
	//利用沙盒根目录拼接”Caches”字符串
	//利用NSSearchPathForDirectoriesInDomains函数(将函数的第2个参数改为：NSCachesDirectory即可)
	
	//Library/Preference：通过NSUserDefaults类存取该目录下的设置信息



## XML属性列表（plist）归档 ##
>- 属性列表是一种XML格式的文件，拓展名为plist
>- 如果对象是NSString、NSDictionary、NSArray、NSData、NSNumber等类型，就可以使用writeToFile:atomically:方法直接将对象写到属性列表文件中

		//TODO1: 获取filepath
	    [data writeToFile:filepath atomically:YES];  //写
        NSArray *data = [NSArray arrayWithContentsOfFile:filepath]; //读

## 偏好设置 ##
>- 类似于Android中的SharedPreference
>- 主要用来保存一些应用的设置，比如用户名，密码，登录状态等等
>- 每个应用都有个NSUserDefaults实例，通过它来存取偏好设置
>- 本质保存格式其实也是.plist文件

	//保存
	NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults]; //获取应用的偏好设置文件
	[defaults setObject:@"suixin" forKey:@"username"];
	[defaults setFloat:18.0f forKey:@"text_size"];
	[defaults setBool:YES forKey:@"auto_login"];

	[defaults synchornize];// 同步到文件中


	//读取
	NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
	NSString *username = [defaults stringForKey:@"username"];
	float textSize = [defaults floatForKey:@"text_size"];
	BOOL autoLogin = [defaults boolForKey:@"auto_login"];

>- 为何需要同步
>    - UserDefaults设置数据时，不是立即写入，而是根据时间戳定时地把缓存中的数据写入本地磁盘。
>    所以调用了set方法之后数据有可能还没有写入磁盘应用程序就终止了。出现以上问题，可以通过调用synchornize方法强制写入

##  NSKeyedArchiver归档(NSCoding) ##
>- 一个对象如果想使用NSKeyedArchiver来进行归档(保存)，必须遵守了NSCoding协议
>- NSString、NSDictionary、NSArray、NSData、NSNumber都已经遵守这个协议
>- NSCoding协议有2个方法：
>     - encodeWithCoder:
>       每次归档对象时，都会调用这个方法。一般在这个方法里面指定如何归档对象中的每个实例变量，可以使用encodeObject:forKey:方法归档实例变量
>     - initWithCoder:
>       每次从文件中恢复(解码)对象时，都会调用这个方法。一般在这个方法里面指定如何解码文件中的数据为对象的实例变量，可以使用decodeObject:forKey方法解码实例变量

	//归档一个NSArray对象到Documents/array.archive
	NSArray *array = [NSArray arrayWithObjects:@”a”,@”b”,nil];
	[NSKeyedArchiver archiveRootObject:array toFile:path];
	
	//读档
	NSArray *array = [NSKeyedUnarchiver unarchiveObjectWithFile:path];


	//对一个自定义对象Person进行归档
	@implementation Person
	- (void)encodeWithCoder:(NSCoder *)encoder {
	    [encoder encodeObject:self.name forKey:@"name"];
	    [encoder encodeInt:self.age forKey:@"age"];
	    [encoder encodeFloat:self.height forKey:@"height"];
	}
	- (id)initWithCoder:(NSCoder *)decoder {
	    self.name = [decoder decodeObjectForKey:@"name"];
	    self.age = [decoder decodeIntForKey:@"age"];
	    self.height = [decoder decodeFloatForKey:@"height"];
	    return self;
	}
	- (void)dealloc {
	    [super dealloc];
	    [_name release];
	}
	@end

	//归档(编码)
	Person *person = [[[Person alloc] init] autorelease];
	person.name = @"MJ";
	person.age = 27;
	person.height = 1.83f;
	[NSKeyedArchiver archiveRootObject:person toFile:path];
	恢复(解码)
	Person *person = [NSKeyedUnarchiver unarchiveObjectWithFile:path];


	//NSData-归档2个Person对象到同一文件中
	// 新建一块可变数据区
	NSMutableData *data = [NSMutableData data];
	// 将数据区连接到一个NSKeyedArchiver对象
	NSKeyedArchiver *archiver = [[[NSKeyedArchiver alloc] initForWritingWithMutableData:data] autorelease];
	// 开始存档对象，存档的数据都会存储到NSMutableData中
	[archiver encodeObject:person1 forKey:@"person1"];
	[archiver encodeObject:person2 forKey:@"person2"];
	// 存档完毕(一定要调用这个方法)
	[archiver finishEncoding];
	// 将存档的数据写入文件
	[data writeToFile:path atomically:YES];

	//恢复（解码）
	// 从文件中读取数据
	NSData *data = [NSData dataWithContentsOfFile:path];
	// 根据数据，解析成一个NSKeyedUnarchiver对象
	NSKeyedUnarchiver *unarchiver = [[NSKeyedUnarchiver alloc] initForReadingWithData:data];
	Person *person1 = [unarchiver decodeObjectForKey:@"person1"];
	Person *person2 = [unarchiver decodeObjectForKey:@"person2"];
	// 恢复完毕
	[unarchiver finishDecoding];










