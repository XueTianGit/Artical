title: Android进阶-如何避免频繁修改UI代码
date: 2016/3/3 21:04:11                
categories: Android
---
# Android进阶-如何避免频繁修改UI代码 #

- 问题：
> 当我们的UI展示代码和业务代码混在一起时， 如果频繁修改UI需求，
> 那么的话我们就要频繁修改UI代码！但是这并不是一件有趣的事情！！！而是一件会使人产生暴力倾向的事情！！

那么如何解决呢？
>- 将业务代码中的需要展示UI的代码去掉，提供一个UI接口
>- 外部调用业务方法时，实现这个接口，即如何刷新UI界面

例如:

	public class MusicPlayer{

		public interface ProcessUI{
			showProcessUI(int process, int totalProcess);
		}
		//这段代码，会得到一个音乐或者视频的进度，并展示进度
		public void showProcess(ProcessUI processUi){
			/*得到进度 process*/
			processUI.showProcessUI(process, totalProcess);/*UI显示交给外面处理*/
			......
						
		} 
	}

	//外部调用
	musicPlayer.showProcess(new ProcessUI{
		@override
		showProcessUI(int process, int totalProcess){
			/*显示进度的UI界面*/
			/*在这里，你再怎么改UI代码，也就在这里， 不用老是跑到
			  业务方法中取，改来改去！！！
			*/
		}
	})；
