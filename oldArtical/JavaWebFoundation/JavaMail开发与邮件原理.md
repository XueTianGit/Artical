title: JavaWebFoundation-JavaMail开发与邮件原理
date: 2016/3/1 9:18:40               
categories: JavaWebFoundation
---

# JavaMail开发与邮件原理 #

## 基本概念  ##
- 邮件服务器：
> 要在Internet上提供电子邮件功能，必须有专门的电子邮件服务器，例如sina， 163等他们都有自己的邮件服务器。
> 这些服务器类似于现实生活中的邮局，它主要负责接收用户投递过来的邮件，并把邮件投递到接收者的电子邮箱。

- 电子邮箱（Email地址）：
> 电子邮箱的获得需要在邮件服务器上进行申请，电子邮箱其实就是用户在邮件服务器上申请的一个账号， 这个账号会有一定的空间，用来发送邮件和保存别人发送过来的电子邮件。

- 发邮件是从客户端把邮件发送到邮件服务器，收邮件是把邮件服务器的邮件下载到客户端。

## 邮件传送协议 ##
>  常见的邮件协议包括：

- SMTP：简单邮件传输协议，用于发送电子邮件的传输协议；   SMTP服务器一般工作在25号端口
- POP3：用于接收电子邮件的标准协议；                   POP3一般工作在110端口
- IMAP：互联网消息协议，是POP3的替代协议。

> 其实每个邮件服务器都由SMTP服务器和POP3服务器构成，其中SMTP服务器负责发邮件的请求，而POP3负责收邮件的请求。

![]http://7xrbxa.com1.z0.glb.clouddn.com/JavaWebFoundation%E9%82%AE%E4%BB%B61.png)

> 当然，有时我们也会使用163的账号，向126的账号发送邮件。
> 这时邮件是发送到126的邮件服务器，而对于163的邮件服务器是不会存储这封邮件的。

![](http://7xrbxa.com1.z0.glb.clouddn.com/JavaWebFoundation%E9%82%AE%E4%BB%B62.png)

### 邮件服务器名称 ###
- smtp服务器的端口号为25，服务器名称为smtp.xxx.xxx。
- pop3服务器的端口号为110，服务器名称为pop3.xxx.xxx。
- 例如：
>- 163：smtp.163.com和pop3.163.com；
>- 126：smtp.126.com和pop3.126.com；
>- qq：smtp.qq.com和pop3.qq.com；
>- sohu：smtp.sohu.com和pop3.sohu.com；
>- sina：smtp.sina.com和pop3.sina.com。


## telnet收发邮件 ##

### 发邮件 ###

这个示例实在命令窗口简单示范的，
我们在进行登录时输入的用户名和密码必须是经过base64算法加密后的。

	telnet  smtp.163.com 25  //连接163的smtp服务器：
							 //连接成功后需要如下步骤才能发送邮件
	ehlo xxx                 //1. 与服务器打招呼：ehlo你的名字
	 
	auth login 				 //2. 发出登录请求：
	 
	aXRjYXN0X2N4ZkAxNjMuY29t    //3	输入加密后的邮箱名：(itcast_cxf@163.com)aXRjYXN0X2N4ZkAxNjMuY29t
	aXRjYXN04				   //4. 输入加密后的邮箱密码：(itcast)aXRjYXN0
	 
	from：mail from:<suixin_send@163.com> //5. 输入谁来发送邮件
	 
	to：rcpt to:<suixin_receive@126.com>  //6. 输入把邮件发给谁，
	 
	data                                //7. 发送填写数据请求：data
	 						
	from:<suixin_send@163.com>
	to:<suixin_receive@sina.com>
	subject: 测试                    //8. 开始输入数据，数据包含：from、to、subject，以及邮件内容，如果输入结束后，以一个“.”为一行，表示输入结束：
	
	把课件不小心删了， 真的烦！！！！     //NT:在标题和邮件正文之间要有一个空行！当要退出时，一定要以一个“.”为单行，表示输入结束。
	.
	 
	quit     　　//9. 最后一步：

### 收邮件 ###

1　telnet收邮件的步骤
pop3无需使用Base64加密！！！

> 收邮件连接的服务器是pop3.xxx.com，pop3协议的默认端口号是110。
> 请注意！这与发邮件完全不同。如果你在163有邮箱账户，那么你想使用telnet收邮件，
> 需要连接的服务器是pop3.163.com。

- 连接pop3服务器：telnet pop3.163.com 110
- user命令：user 用户名，例如：user itcast_cxf@163.com；
- pass命令：pass 密码，例如：pass itcast；
- stat命令：stat命令用来查看邮箱中邮件的个数，所有邮件所占的空间；
- list命令：list命令用来查看所有邮件，或指定邮件的状态，例如：list 1是查看第一封邮件的大        小，list是查看邮件列表，即列出所有邮件的编号，及大小；
- retr命令：查看指定邮件的内容，例如：retr 1#是查看第一封邮件的内容；
- dele命令：标记某邮件为删除，但不是马上删除，而是在退出时才会真正删除；
- quit命令：退出！如果在退出之前已经使用dele命令标记了某些邮件，那么会在退出是删除它们。

	example:
	
	telent pop3.126.com 110
	
	user aaa
	pass 123
	
	stat 
	
	retr 2   //接收第二封邮件
	
	quit   

## JavaMail ##

### JavaMail概述 ###
Java Mail是由SUN公司提供的专门针对邮件的API，主要Jar包：mail.jar、activation.jar。


>- 在使用MyEclipse创建web项目时，需要小心！如果只是在web项目中使用java mail是没有什么问题的，
>  发布到Tomcat上运行一点问题都没有！
>- 但是如果是在web项目中写测试那就出问题了。
>- 在MyEclipse中，会自动给web项目导入javax.mail包中的类，但是不全（其实是只有接口，而没有接口
>  的实现类），所以只靠MyEclipse中的类是不能运行java mail项目的，但是如果这时你再去自行导入
>  mail.jar时，就会出现冲突。
>- 处理方案：到下面路径中找到javaee.jar文件，把javax.mail删除！！！  再导自己的

### JavaMail中主要类 ###

>- Session：表示会话，即客户端与邮件服务器之间的会话！想获得会话需要给出账户和密码，当然还要给出服务    >  器名称。在邮件服务中的Session对象，就相当于连接数据库时的Connection对象。
>- MimeMessage：表示邮件类，它是Message的子类。它包含邮件的主题（标题）、内容，收件人地址、发件人地> >  址，还可以设置抄送和暗送，甚至还可以设置附件。
>- Transport：用来发送邮件。它是发送器！

### 　JavaMail之Hello World ###

第一步：获得Session
Session session = Session.getInstance(Properties prop, Authenticator auth); 
其中prop需要指定两个键值，一个是指定服务器主机名，另一个是指定是否需要认证！
Properties prop = new Properties();
prop.setProperty(“mail.host”, “smtp.163.com”);//设置服务器主机名
prop.setProperty(“mail.smtp.auth”, “true”);//设置需要认证
其中Authenticator是一个接口表示认证器，即校验客户端的身份。我们需要自己来实现这个接口，实现这个接口需要使用账户和密码。
Authenticator auth = new Authenticator() {
    public PasswordAuthentication getPasswordAuthentication () {
        new PasswordAuthentication(“itcast_cxf”, “itcast”);//用户名和密码
}
};
通过上面的准备，现在可以获取得Session对象了：
Session session = Session.getInstance(prop, auth);

第二步：创建MimeMessage对象
创建MimeMessage需要使用Session对象来创建：
MimeMessage msg = new MimeMessage(session);
然后需要设置发信人地址、收信人地址、主题，以及邮件正文。
msg.setFrom(new InternetAddress(“itcast_cxf@163.com”));//设置发信人
msg.addRecipients(RecipientType.TO, “itcast_cxf@qq.com,itcast_cxf@sina.com”);//设置多个收信人
msg.addRecipients(RecipientType.CC, “itcast_cxf@sohu.com,itcast_cxf@126.com”);//设置多个抄送
msg.addRecipients(RecipientType.BCC, ”itcast_cxf@hotmail.com”);//设置暗送
msg.setSubject(“这是一封测试邮件”);//设置主题（标题）
msg.setContent(“当然是hello world!”, “text/plain;charset=utf-8”);//设置正文

第三步：发送邮件
Transport.send(msg);//发送邮件


public void test() throws Exception {
		/*
		 * 1. 得到session
		 */
		Properties props = new Properties();
		props.setProperty("mail.host", "smtp.163.com");
		props.setProperty("mail.smtp.auth", "true");
		
		Authenticator auth = new Authenticator() {
			@Override
			protected PasswordAuthentication getPasswordAuthentication() {
				return new PasswordAuthentication("itcast_cxf", "itcast");
			}
		};
		
		Session session = Session.getInstance(props, auth);
		
		/*
		 * 2. 创建MimeMessage
		 */
		MimeMessage msg = new MimeMessage(session);
		msg.setFrom(new InternetAddress("itcast_cxf@163.com"));//设置发件人
		msg.setRecipients(RecipientType.TO, "itcast_cxf@126.com");//设置收件人
		msg.setRecipients(RecipientType.CC, "itcast_cxf@sohu.com");//设置抄送
		msg.setRecipients(RecipientType.BCC, "itcast_cxf@sina.com");//设置暗送
		
		msg.setSubject("这是来自ITCAST的测试邮件");
		msg.setContent("这就是一封垃圾邮件！", "text/html;charset=utf-8");
		
		/*
		 * 3. 发
		 */
		Transport.send(msg);
	}



### JavaMail发送带有附件的邮件（ ###
一封邮件可以包含正文、附件N个，所以正文与N个附件都是邮件的一个部份。

上面的hello world案例中，只是发送了带有正文的邮件！所以在调用setContent()方法时直接设置了正文，如果想发送带有附件邮件，那么需要设置邮件的内容为MimeMultiPart。
MimeMulitpart parts = new MimeMulitpart();//多部件对象，可以理解为是部件的集合
msg.setContent(parts);//设置邮件的内容为多部件内容。
然后我们需要把正文、N个附件创建为“主体部件”对象（MimeBodyPart），添加到MimeMuiltPart中即可。
MimeBodyPart part1 = new MimeBodyPart();//创建一个部件
part1.setCotnent(“这是正文部分”, “text/html;charset=utf-8”);//给部件设置内容
parts.addBodyPart(part1);//把部件添加到部件集中。

下面我们创建一个附件：
MimeBodyPart part2 = new MimeBodyPart();//创建一个部件
part2.attachFile(“F:\\a.jpg”);//设置附件
part2.setFileName(“hello.jpg”);//设置附件名称
parts.addBodyPart(part2);//把附件添加到部件集中

注意，如果在设置文件名称时，文件名称中包含了中文的话，那么需要使用MimeUitlity类来给中文编码：
part2.setFileName(MimeUitlity.encodeText(“美女.jpg”));

![](http://7xrbxa.com1.z0.glb.clouddn.com/JavaWebFoundation%E9%82%AE%E4%BB%B63.png)


	public void test2() throws Exception {
		/*
		 * 1. 得到session
		 */
		Properties props = new Properties();
		props.setProperty("mail.host", "smtp.163.com");
		props.setProperty("mail.smtp.auth", "true");
		
		Authenticator auth = new Authenticator() {
			@Override
			protected PasswordAuthentication getPasswordAuthentication() {
				return new PasswordAuthentication("itcast_cxf", "itcast");
			}
		};
		
		Session session = Session.getInstance(props, auth);
		
		/*
		 * 2. 创建MimeMessage
		 */
		MimeMessage msg = new MimeMessage(session);
		msg.setFrom(new InternetAddress("itcast_cxf@163.com"));//设置发件人
		msg.setRecipients(RecipientType.TO, "itcast_cxf@126.com");//设置收件人
		
		msg.setSubject("这是来自ITCAST的测试邮件有附件");
		
		
		////////////////////////////////////////////////////////
		/*
		 * 当发送包含附件的邮件时，邮件体就为多部件形式！
		 * 1. 创建一个多部件的部件内容！MimeMultipart
		 *   MimeMultipart就是一个集合，用来装载多个主体部件！
		 * 2. 我们需要创建两个主体部件，一个是文本内容的，另一个是附件的。
		 *   主体部件叫MimeBodyPart
		 * 3. 把MimeMultipart设置给MimeMessage的内容！
		 */
		MimeMultipart list = new MimeMultipart();//创建多部分内容
		
		// 创建MimeBodyPart
		MimeBodyPart part1 = new MimeBodyPart();
		// 设置主体部件的内容
		part1.setContent("这是一封包含附件的垃圾邮件", "text/html;charset=utf-8");
		// 把主体部件添加到集合中
		list.addBodyPart(part1);
		
		
		// 创建MimeBodyPart
		MimeBodyPart part2 = new MimeBodyPart();
		part2.attachFile(new File("F:/f/白冰.jpg"));//设置附件的内容
		part2.setFileName(MimeUtility.encodeText("大美女.jpg"));//设置显示的文件名称，其中encodeText用来处理中文乱码问题
		list.addBodyPart(part2);
		
		msg.setContent(list);//把它设置给邮件作为邮件的内容。
		
		
		////////////////////////////////////////////////////////
		
		/*
		 * 3. 发
		 */
		Transport.send(msg);		
	}
	
	@Test
	public void test3() throws Exception {
		/*
		 * 1. 得到session
		 */
		Session session = MailUtils.createSession("smtp.163.com", 
				"itcast_cxf", "itcast");
		/*
		 * 2. 创建邮件对象
		 */
		Mail mail = new Mail("itcast_cxf@163.com",
				"itcast_cxf@126.com,itcast_cxf@sina.com",
				"不是垃圾邮件能是什么呢？", "这里是正文");
		
		/*
		 * 创建两个附件对象
		 */
		AttachBean ab1 = new AttachBean(new File("F:/f/白冰.jpg"), "小美女.jpg");
		AttachBean ab2 = new AttachBean(new File("F:/f/big.jpg"), "我的羽绒服.jpg");
		
		// 添加到mail中
		mail.addAttach(ab1);
		mail.addAttach(ab2);
		
		/*
		 * 3. 发送
		 */
		MailUtils.send(session, mail);
	}





