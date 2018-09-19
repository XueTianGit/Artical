title: Android进阶-WebView 
date: 2016/3/3 21:11:02                           
categories: Android
---

# Android进阶-WebView #

>- The WebView class is an extension of Android's View class that allows you to display web pages as a part of your activity layout. 
>- It does not include any features of a fully developed web browser, such as navigation controls or an address bar. All that WebView does, 
>- by default, is show a web page.
>- 即我们可以将WebView看做一个内嵌的浏览器放在我们的应用中，在我们想使用WebView的地方，直接在布局文件中设置一下就可以，然后就可以像使用普通UI控件一样使用它。

- 下面来简单的看一下WebView的功能

>- 最简单的加载一个网页

	webView.loadUri(String uri)   //加载一个网页
>- 跳转到，前一个加载的网页，或者后面的网页
	webView.goBackk();
	webView.goForward();

>- 对于WebView的功能设置， 可以通过 WebSettings 对象来完成
>- 进而是WebView支持更多的功能
	WebSettings webSettings =   mWebView .getSettings(); 

	setJavaScriptEnabled(true);  //支持js
	setJavaScriptCanOpenWindowsAutomatically(true); //支持通过JS打开新窗口 
	
	setPluginsEnabled(true);  //支持插件 
	
	setUseWideViewPort(false);  //将图片调整到适合webview的大小 
	
	setSupportZoom(true);  //支持缩放 
	setBuiltInZoomControls(true); //设置支持缩放 
	
	setLayoutAlgorithm(LayoutAlgorithm.SINGLE_COLUMN); //支持内容重新布局  
	
	supportMultipleWindows();  //多窗口 
	
	setCacheMode(WebSettings.LOAD_CACHE_ELSE_NETWORK);  //关闭webview中缓存 
	
	setAllowFileAccess(true);  //设置可以访问文件 
	
	setNeedInitialFocus(true); //当webview调用requestFocus时为webview设置节点
	
	setLoadWithOverviewMode(true); // 缩放至屏幕的大小（打开网页时，使网页自动适应屏幕）
	
	setLoadsImagesAutomatically(true);  //支持自动加载图片      


>- WebViewClient对象。 这个对象类似与一个监听器，我们可以使用它，在WebView加载网页时做一些事情
>- 这里只列举一些， 其他还有很多
	mWebView.setWebViewClient(new WebViewClient() {
			/**
			 * 网页开始加载
			 */
			@Override
			public void onPageStarted(WebView view, String url, Bitmap favicon) {
				super.onPageStarted(view, url, favicon);
				System.out.println("网页开始加载");
			}

			/**
			 * 网页加载结束
			 */
			@Override
			public void onPageFinished(WebView view, String url) {
				super.onPageFinished(view, url);
				System.out.println("网页开始结束");
			}

			/**
			 * 所有跳转的链接都会在此方法中回调
			 */
			@Override
			public boolean shouldOverrideUrlLoading(WebView view, String url) {
				// tel:110
				System.out.println("跳转url:" + url);
				view.loadUrl(url);   //不调用系统浏览器， 继续在本WebView中加载该网页
				return true;
				// return super.shouldOverrideUrlLoading(view, url);
			}
		});

		
>- WebChromeClient类似于WebViewClient， 我们也可以通过它，知道一些事情

	mWebView.setWebChromeClient(new WebChromeClient() {

			/**
			 * 进度发生变化
			 */
			@Override
			public void onProgressChanged(WebView view, int newProgress) {
				System.out.println("加载进度:" + newProgress);
				super.onProgressChanged(view, newProgress);
			}

			/**
			 * 获取网页标题
			 */
			@Override
			public void onReceivedTitle(WebView view, String title) {
				System.out.println("网页标题:" + title);
				super.onReceivedTitle(view, title);
			}
		});

 