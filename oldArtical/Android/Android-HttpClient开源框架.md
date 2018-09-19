title: Android-网络框架
date: 2016/2/29 8:09:49    
categories: Android
---
# Android-网络框架 #

## HttpClient框架 ##
>- HttpClient是Apache Jakarta Common下的子项目，用来提供高效的、最新的、功能丰富的支持HTTP协议的客户端编程工具包，并且它支持HTTP协议最新的版本和建议。
>- 他被内置在Android中，我们在编写android代码时可以直接使用。

### 简单使用范例  ###

	//get方式
    public void get(View v){
    	EditText et_name = (EditText) findViewById(R.id.et_name);
    	EditText et_pass = (EditText) findViewById(R.id.et_pass);
    	
    	final String name = et_name.getText().toString();
    	final String pass = et_pass.getText().toString();
    	
    	Thread t = new Thread(){
    		@Override
    		public void run() {
				//对于中文参数，应进行URL编码
    			String path = "http://192.168.13.13/Web/servlet/CheckLogin?name=" + URLEncoder.encode(name) + "&pass=" + pass;

    	    	//使用httpClient框架做get方式提交
    	    	//1.创建HttpClient对象
    	    	HttpClient hc = new DefaultHttpClient();
    	    	
    	    	//2.创建httpGet对象，构造方法的参数就是网址
    	    	HttpGet hg = new HttpGet(path);
    	    	
    	    	//3.使用客户端对象，把get请求对象发送出去
    	    	try {
    				HttpResponse hr = hc.execute(hg);
    				//拿到响应头中的状态行
    				StatusLine sl = hr.getStatusLine();
    				if(sl.getStatusCode() == 200){
    					//拿到响应头的实体
    					HttpEntity he = hr.getEntity();
    					//拿到实体中的内容，其实就是服务器返回的输入流
    					InputStream is = he.getContent();
    					
						/*doSomething*/
    					
    				}
    			} catch (Exception e) {
    				// TODO Auto-generated catch block
    				e.printStackTrace();
    			}
    		}
    	};
    	t.start();
    	
    }

	public void post(View v){
    	EditText et_name = (EditText) findViewById(R.id.et_name);
    	EditText et_pass = (EditText) findViewById(R.id.et_pass);
    	
    	final String name = et_name.getText().toString();
    	final String pass = et_pass.getText().toString();
    	
    	Thread t = new Thread(){
    		@Override
    		public void run() {
    			String path = "http://192.168.13.13/Web/servlet/CheckLogin";
    	    	//1.创建客户端对象
    	    	HttpClient hc = new DefaultHttpClient();
    	    	//2.创建post请求对象
    	    	HttpPost hp = new HttpPost(path);
    	    	
    	    	//封装form表单提交的数据
    	    	BasicNameValuePair bnvp = new BasicNameValuePair("name", name);
    	    	BasicNameValuePair bnvp2 = new BasicNameValuePair("pass", pass);
    	    	List<BasicNameValuePair> parameters = new ArrayList<BasicNameValuePair>();
    	    	//把BasicNameValuePair放入集合中
    	    	parameters.add(bnvp);
    	    	parameters.add(bnvp2);
    	    	
    	    	try {
    	    		//要提交的数据都已经在集合中了，把集合传给实体对象
    		    	UrlEncodedFormEntity entity = new UrlEncodedFormEntity(parameters, "utf-8");
    		    	//设置post请求对象的实体，其实就是把要提交的数据封装至post请求的输出流中
    		    	hp.setEntity(entity);
    		    	//3.使用客户端发送post请求
    				HttpResponse hr = hc.execute(hp);
    				if(hr.getStatusLine().getStatusCode() == 200){
    					InputStream is = hr.getEntity().getContent();
    					/*doSomething*/
    				}
    			} catch (Exception e) {
    				// TODO Auto-generated catch block
    				e.printStackTrace();
    			}
    		}
    	};
    	t.start();	
    }

## 异步HttpClient框架 ##

>- 使用HttpClient框架，可以使我们的网络编程的代码更加清晰，但是我们实际写的代码好像并没有减少，
>- 程序员，有时懒一点总是好的，我们可以使用第三方异步HttpClient框架，他其实只不过是对HttpClient框架的封装。 
>- 可以去github上找一下这个框架，  android-async-http-master.zip。

### 简单使用 ###
> 代码变少的许多

    public void get(View v){
    	EditText et_name = (EditText) findViewById(R.id.et_name);
    	EditText et_pass = (EditText) findViewById(R.id.et_pass);
    	
    	final String name = et_name.getText().toString();
    	final String pass = et_pass.getText().toString();
    	String url = "http://192.168.13.13/Web/servlet/CheckLogin?name=" + URLEncoder.encode(name) + "&pass=" + pass;
    	//创建异步httpclient
    	AsyncHttpClient ahc = new AsyncHttpClient();
    	
    	//发送get请求提交数据
    	ahc.get(url, new MyResponseHandler());
    }
    
    public void post(View v){
    	EditText et_name = (EditText) findViewById(R.id.et_name);
    	EditText et_pass = (EditText) findViewById(R.id.et_pass);
    	
    	final String name = et_name.getText().toString();
    	final String pass = et_pass.getText().toString();
    	String url = "http://192.168.13.13/Web/servlet/CheckLogin";
    	
    	//创建异步httpclient
    	AsyncHttpClient ahc = new AsyncHttpClient();
    	
    	//发送post请求提交数据
    	//把要提交的数据封装至RequestParams对象
    	RequestParams params = new RequestParams();
    	params.add("name", name);
    	params.add("pass", pass);
    	ahc.post(url, params, new MyResponseHandler());
    }
    
    class MyResponseHandler extends AsyncHttpResponseHandler{

    	//请求服务器成功时，此方法调用
		@Override
		public void onSuccess(int statusCode, Header[] headers,
				byte[] responseBody) {

			Toast.makeText(MainActivity.this, new String(responseBody), 0).show();
			
		}

		//请求失败此方法调用
		@Override
		public void onFailure(int statusCode, Header[] headers,
				byte[] responseBody, Throwable error) {
			Toast.makeText(MainActivity.this, "请求失败", 0).show();
			
		}
    	
    }
