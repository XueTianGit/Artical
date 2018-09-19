title: Android进阶-利用BitmapUtils使ListView在快速滑动时不加载图片
date: 2016/3/3 21:09:38                       
categories: Android
---

# Android进阶-利用BitmapUtils使ListView在快速滑动时不加载图片 #

>- 如果我们的ListView的条目中含有图片的话， 那么，在ListView快速滑动时是没有必要去加载图片的
>- 我们可以使用BitmapUtils的PauseOnScrollListener来完成这个任务

		// 第二个参数 慢慢滑动的时候是否加载图片 false  加载   true 不加载
		//  第三个参数  飞速滑动的时候是否加载图片  true 不加载 
		listView.setOnScrollListener(new PauseOnScrollListener(bitmapUtils, false, true));
		bitmapUtils.configDefaultLoadingImage(R.drawable.ic_default);  // 设置如果图片加载中显示的图片
        bitmapUtils.configDefaultLoadFailedImage(R.drawable.ic_default);// 加载失败显示的图片

>- 不要担心，因为new PauseOnScrollListener而导致你不能再监听listview滑动的问题
>- BitmapUtils 提供了PauseOnScrollListener的多种形式
>- public PauseOnScrollListener(TaskHandler taskHandler, boolean pauseOnScroll, boolean pauseOnFling, OnScrollListener customListener)