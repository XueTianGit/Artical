title: Oracle-卸载
date: 2016/4/19 8:45:55  
categories: Database
---

# Oracle-卸载 #

用Oracle自带的卸载程序不能从根本上卸载Oracle，从而为下次的安装留下隐患，那么怎么才能完全卸载Oracle呢？
那就是直接注册表清除，步骤如下： 
	
	1、 开始－>设置－>控制面板－>管理工具－>服务 
	    停止所有Oracle服务。 
	
	2、 开始－>程序－>Oracle - OraDb11g_home1－>Oracle安装产品－> Universal Installer 
	    卸装所有Oracle产品，但Universal Installer本身不能被删除[如果第二步执行失败，跳到第三步，大部份第二步是失败的] 
	
	3、 运行regedit，选择HKEY_LOCAL_MACHINE\SOFTWARE\ORACLE，按del键删除这个入口。 
	
	4、 运行regedit，删除以下这三个位置中的所有Oracle入口。
		HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\【下】所有Oracle删除
		HKEY_LOCAL_MACHINE\SYSTEM\ControlSet002\Services\【下】所有Oracle删除
		HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\【下】所有Oracle删除
	
	5、 运行regedit， 
	   HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Eventlog\Application\【下】所有Oracle删除， 
	   删除所有Oracle入口。 
	
	6、 开始－>设置－>控制面板－>系统－>高级－>环境变量 
	    删除环境变量CLASSPATH和PATH中有关Oracle的设定 
	
	7、 从桌面上、STARTUP（启动）组、程序菜单中，删除所有有关Oracle的组和图标 
	
	8、 删除e:/oracleDB目录
	
	9、 【重新启动计算机】，重起后才能完全删除Oracle所在目录 
	
	10、 删除与Oracle有关的文件，选择Oracle所在的缺省目录C:\Oracle，删除这个入 
	     口目录及所有子目录，并从Windows目录（一般为C:\WINDOWS）下删除oralce文件等等。 
	
	11、 在运行框中输入“win.ini”，回车。WIN.INI文件中若有[ORACLE]的标记段，删除该段 
	
	12、 【如有必要】，删除所有Oracle相关的ODBC的DSN 
	
	13、 到事件查看器中，删除Oracle相关的日志 

说明： 
如果有个别DLL文件无法删除的情况，则不用理会，重新启动，开始新的安装， 
安装时，选择一个新的目录，则，安装完毕并重新启动后，老的目录及文件就可以删除掉了


**摘抄自 传智博客笔记。**



	> 从卸载Oracle来看， 许多删不干净的软件，可能都是因为没有把注册表删干净， 
	> 而进入注册表主要删除的有这几个目录下的软件相关的文件夹
	> HKEY_LOCAL_MACHINE\SOFTWARE\【下】
	> HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\【下】
	> HKEY_LOCAL_MACHINE\SYSTEM\ControlSet002\Services\【下】
	> HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\【下】
	> HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Eventlog\Application\【下】
	> 还有就是要把C盘在相关文件夹也给干掉





