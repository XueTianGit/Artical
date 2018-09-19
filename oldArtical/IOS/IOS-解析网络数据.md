title: IOS-解析网络数据
date: 2016/3/9 8:50:08             
categories: IOS
---
# IOS-解析网络数据 #

>这里主要来看一下IOS中对JSON和XML解析相关的API

## JSON ##

### 解析JSON ###
>- 从iOS 5开始，使用NSJSONSerialization对JSON解析
>- 其他常见的三种JSON解析第三方库：
>    - SBJson	因为API简单易用，可能还会有一些应用中留存
>    - JSONKit	JSONKit的开发者称：JSONKit的性能优于苹果
>    - TouchJson
>- 可以把json转成字典或者数组

    /**
     技巧:如果看到枚举类型的第一个选项数值不为0,意味着如果使用0,就什么特殊功能都不做,性能最好!
     options可取下列枚举值：
     NSJSONReadingMutableContainers = 1 根节点可变   //返回的id类型为可变的字典
     NSJSONReadingMutableLeaves     = 2 子节点可变
     NSJSONReadingAllowFragments    = 4 默认情况下只能反序列化NSArray或者NSDictionary,可以是其他类型
     */
	//这里data是服务器发过来的二进制json数据
	//id dict = [NSJSONSerialization JSONObjectWithData:data options:NSJSONReadingMutableContainers error:NULL];
	//这里你知道这个id到底是什么类型(当然也可能是一个NSArray)
    id dict = [NSJSONSerialization JSONObjectWithData:data options:0 error:NULL];  //使用0就给你返回一个装好json数据的字典
	//后续就这个字典就得你自己来转成模型数据了，(可以使用KVC)

当然也可以把字典或者数组给转成json数据

	[NSJSONSerialization dataWithJSONObject:array options:0 error:NULL];

## XML ##

### NSXMLParser解析XML ###
>- NSXMLParser是SAX方法解析
>- 解析过程
>    - 实例化NSXMLParser，传入从服务器接收的XML数据
>    - 定义解析器代理
>    - 解析器解析
>    - 通过解析代理方法完成XML数据的解析

NSXMLParser解析代理方法：

	// 1. 开始解析XML文档
	-(void)parserDidStartDocument:
	
	// 2. 开始解析某个元素，会遍历整个XML，识别元素节点名称
	-(void)parser:didStartElement:namespaceURI:qualifiedName:attributes:
	
	// 3. 文本节点，得到文本节点里存储的信息数据，对于大数据可能会接收多次！为了节约内存开销
	-(void)parser:foundCharacters:

	// 4. 结束某个节点，存储从parser:foundCharacters:方法中获取到的信息
	-(void)parser:didEndElement:namespaceURI:qualifiedName:
	
	注意：在解析过程中，2、3、4三个方法会不停的重复执行，直到遍历完成为止
	
	// 5. 解析XML文档结束
	-(void)parserDidEndDocument:
	
	// 6. 解析出错
	-(void)parser:parseErrorOccurred:

