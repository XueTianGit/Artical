title: JavaWebFoundation-Lucene-基础二
date: 2016/3/1 9:18:40               
categories: JavaWebFoundation
---

# Lucene-基础二 #

lucene各种版本jar：http://archive.apache.org/dist/lucene/java/

## 索引库的优化 ##

- 由于，在默认情况下，向索引库中增加一个Document对象时，索引库自动会添加一个扩展名叫*.cfs的二进制压缩文件，
- 如果向索引库中存Document对象过多，那么*.cfs也会不断增加，同时索引库的容量也会不断增加，影响索引库的大小。

> 因此，我们有必要对.cfs的问题进行优化。
> 还有就是，我们在从索引库中操作数据时，默认是程序会去访问硬盘，这样速度是非常慢的，因此，对于索引库操作数据的速度
> 也是需要优化的。

- 优化方案：
> 合并cfs文件，合并后的cfs文件是二进制压缩字符，  -> 能解决是的文件大小和数量的问题 

	a, 调用indexWriter.optimize()  
		当我们向索引中存入document对象后，如果调用了这个方法， 就会
	b，设置合并因子， 这样我们就不必去调用indexWriter.optimize()， 当.cfs文件到达指定数量是，会自动合并。
		indexWriter.setMergeFactor(3);

> 使用RAMDirectory，类似于内存索引库（cache），即把硬盘索引库缓存到内存。这样就能解决读取索引库文件的速度问题。
> 
> 这当然是一种空间换时间的优化方案。 因此启动时加载硬盘中的索引库到内存中的索引库，退出时将内存中的索引库保存到硬盘中的索引库，且内容不能重复。

		Article article = new Article(1,"培训","传智是一家Java培训机构");
		Document document = LuceneUtil.javabean2document(article);
		
		Directory fsDirectory = FSDirectory.open(new File("E:/indexDBDBDBDBDBDBDBDB"));  //硬盘索引库
		Directory ramDirectory = new RAMDirectory(fsDirectory);   //缓存到RAM中
		
		//存取流：   到硬盘的流我们还是要开启的，作用当然是将内存索引库写入硬盘
					到内存的存取流，用他来和索引库交互数据
		IndexWriter fsIndexWriter = new IndexWriter(fsDirectory,LuceneUtil.getAnalyzer(),true,LuceneUtil.getMaxFieldLength());
		IndexWriter ramIndexWriter = new IndexWriter(ramDirectory,LuceneUtil.getAnalyzer(),LuceneUtil.getMaxFieldLength());
		
		ramIndexWriter.addDocument(document);
		ramIndexWriter.close();
		
		fsIndexWriter.addIndexesNoOptimize(ramDirectory);
		fsIndexWriter.close();


## 分词器 Analyzer ##

- 分词器，可以说是采用一种算法，将中英文本中的字符拆分开来，形成词汇，以待用户输入关健字后搜索。
- 分词存在是很有必要的，因为用户输入的搜索的内容是一段文本中的一个关健字，和原始表中的内容有差别，
- 但作为搜索引擎来讲，又得将相关的内容搜索出来，此时就得采用分词器来最大限度匹配原始表中的内容。

>- 分词器的工作流程：
>   - 步一：按分词器拆分出词汇
>   - 步二：去除停用词和禁用词
>   - 步三：如果有英文，把英文字母转为小写，即搜索不分大小写

Lucene内置了许多分词器。
我们可以通过下面这段代码，来测试不同分词器采用的分词策略。
	private static void testAnalyzer(Analyzer analyzer, String text) throws Exception {
		
		System.out.println("当前使用的分词器：" + analyzer.getClass());
		TokenStream tokenStream = analyzer.tokenStream("content",new StringReader(text));   //
		tokenStream.addAttribute(TermAttribute.class);
		while (tokenStream.incrementToken()) {
			TermAttribute termAttribute = tokenStream.getAttribute(TermAttribute.class);
			System.out.println(termAttribute.term());
		}
	}

例如：

	StandardAnalyzer: 以一个字符为分词单位。
	CJKAnalyzer： 两两分词， 左右匹配。
	ChineseAnalyzer：并没什么卵用， 类似于StandardAnalyzer

我们除了使用Luncene内置的分词器外，也可以使用第三方分词器。：

### IKAnalyzer ###
> 对于中文的分词，IKAnalyzer使我们的首选， 听说是依据中科院的研究成果，写出来的，很符合中国汉语的习惯。
> 并且，这个分词器可以配置专有词典，停用词典（过滤词典）。因此有一定的自由度。

#### IKAnalyzer中配置专有词典 ####
- 通过配置专有词典，我们可以是分词器，按我们需求进行分词。
- 步骤：
   - 在根目录下src下引入IKAnalyzer.cfg.xml
   - 书写词典文件
	 - 注意： 词典中每个中文词汇独立占一行， 使用\r\n的DOS方式换行。文件必须保证使用UTF-8格 式保存，并且文件的头部应添加一行。
	 - 词典文件推荐同IKAnalyzer.cfg.xml放在，同一级目录下。
   - IKAnalyzer.cfg.xml的配置
>
		<?xml version="1.0" encoding="UTF-8"?>
		<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">  
		<properties>  
			<comment>IK Analyzer 扩展配置</comment>
			<!-- 用户可以在这里配置自己的扩展字典 --> 
			<entry key="ext_dict">/mydict.dic</entry> 
			<!--用户可以在这里配置自己的扩展停止词字典 -->
			<entry key="ext_stopwords">/surname.dic</entry> 
		</properties>	



## 搜索结果高亮显示 ##
> 我们在百度中搜索时，在搜索结果页面，我们的搜索关键字都是高亮的， 那么在Lucene中如何做到这一点呢？

下面代码, 将与关健字相同的字符用红色显示:

		String keywords = "培训";
		List<Article> articleList = new ArrayList<Article>();
		QueryParser queryParser = new QueryParser(LuceneUtil.getVersion(),"content",LuceneUtil.getAnalyzer());
		Query query = queryParser.parse(keywords);
		IndexSearcher indexSearcher = new IndexSearcher(LuceneUtil.getDirectory());
		TopDocs topDocs = indexSearcher.search(query,1000000);
		
		//
		Formatter formatter = new SimpleHTMLFormatter("<font color='red'>","</font>");
		Scorer scorer = new QueryScorer(query);
		Highlighter highlighter = new Highlighter(formatter,scorer);
		
		for(int i=0;i<topDocs.scoreDocs.length;i++){
			ScoreDoc scoreDoc = topDocs.scoreDocs[i];
			int no = scoreDoc.doc;
			Document document = indexSearcher.doc(no);
			
			//1.把搜索出来的内容加上"<font color='red'>","</font>"
			String highlighterContent = highlighter.getBestFragment(LuceneUtil.getAnalyzer(),"content",document.get("content"));
			//2.然后写到document
			document.getField("content").setValue(highlighterContent);
			//3.再从document中读出来， 内容中的关键字就是加了"<font color='red'>","</font>"。因此在html中显示就是有颜色的
			Article article = (Article) LuceneUtil.document2javabean(document,Article.class);
			articleList.add(article);
		}

		for(Article article : articleList){
			System.out.println(article);
		}
	}

## 搜索结果摘要 ##、
> 还是百度搜索结果页面，在结果页面时，百度只会将搜索内容的部分显示出来，剩余内容用“...”表示。
> 我们只需在调用Highlighter.setTextFragmenter()， 就可实现内容的摘要显示。
> 也可以看出 搜索结果摘要是搜索结果高亮的小弟， 得跟着他混！

看下面代码：

		String keywords = "培训";
		List<Article> articleList = new ArrayList<Article>();
		.....   //与上面的代码相同
		
		//搜索结果摘要！   -》 就是把这两行插到上面搜索结果高亮的代码中去
		Fragmenter fragmenter  = new SimpleFragmenter(4);
		highlighter.setTextFragmenter(fragmenter);
		......  //与上面的代码相同
		for(Article article : articleList){
			System.out.println(article);
		}
	}

## 搜索结果排序 ##
对于搜索结果，我们可以按某个或某些字段高低排序来显示的结果。

>- 影响网站排名的先后的有多种
>   - head/meta/
>   - 网页的标签整洁
>   - 网页执行速度
>   - 采用div+css


> 在lucene中的显示结果次序与相关度得分有关， 即ScoreDoc.score字段
> 默认情况下，Lucene是按相关度得分排序的，得分高排在前，得分低排在后， 如果相关度得分相同，按插入索引库的先后次序排序

我们可以这样操作搜索结果的排序：

1）在Lucene中的设置相关度得分

	IndexWriter indexWriter = new IndexWriter(LuceneUtil.getDirectory(),LuceneUtil.getAnalyzer(),LuceneUtil.getMaxFieldLength());
	document.setBoost(20F);  //设置得分
	indexWriter.addDocument(document);
	indexWriter.close();

2）在Lucene中设置按单个字段排序

	Sort sort = new Sort(new SortField("id",SortField.INT,true));
	TopDocs topDocs = indexSearcher.search(query,null,1000000,sort);
3）在Lucene中设置按多个字段排序

	Sort sort = new Sort(new SortField("count",SortField.INT,true),new SortField("id",SortField.INT,true));
	TopDocs topDocs = indexSearcher.search(query,null,1000000,sort);
> NT： 在多字段排序中，只有第一个字段排序结果相同时，第二个字段排序才有作用	且提倡用数值型排序

##条件搜索 ##

> 用关健字与指定的单列或多例进行匹配的搜索，即在搜索索引库内容时，都是在原始记录表的那些列进行搜索。

- 单字段条件搜索

	QueryParser queryParser = new QueryParser(LuceneUtil.getVersion(),"content",LuceneUtil.getAnalyzer())

2）多字段条件搜索，项目中提倡多字段搜索

	QueryParser queryParser = new MultiFieldQueryParser(LuceneUtil.getVersion(),new String[]{"content","title"},LuceneUtil.getAnalyzer());


**以上代码使用的是lucene3.0.2**
**本文参考传智博客笔记**






