title: Android-对话框
date: 2016/2/29 8:06:57  
categories: Android
---

# Android-对话框 #

下面介绍一下，我们在开发android应用是常用的对话框。

###确定取消对话框

- 使用步骤：
> 创建对话框构建器对象，类似工厂模式

		AlertDialog.Builder builder = new Builder(this);

> 设置标题和正文

    	builder.setTitle("警告");
    	builder.setMessage("若练此功，必先自宫");
> 设置确定和取消按钮

    	builder.setPositiveButton("现在自宫", new OnClickListener() {
			
			@Override
			public void onClick(DialogInterface dialog, int which) {
				// TODO Auto-generated method stub
				Toast.makeText(MainActivity.this, "恭喜你自宫成功，现在程序退出", 0).show();
			}
		});
    	
    	builder.setNegativeButton("下次再说", new OnClickListener() {
			
			@Override
			public void onClick(DialogInterface dialog, int which) {
				// TODO Auto-generated method stub
				Toast.makeText(MainActivity.this, "若不自宫，一定不成功", 0).show();
			}
		});

> 使用构建器创建出对话框对象

    	AlertDialog ad = builder.create();
    	ad.show();


###单选对话框
> 设置标题

		AlertDialog.Builder builder = new Builder(this);
    	builder.setTitle("选择你的性别");
>定义单选选项

    	final String[] items = new String[]{
    			"男", "女", "其他"
    	};

		//-1表示没有默认选择
		//点击侦听的导包要注意别导错
    	builder.setSingleChoiceItems(items, -1, new OnClickListener() {
			
			//which表示点击的是哪一个选项
			@Override
			public void onClick(DialogInterface dialog, int which) {
				Toast.makeText(MainActivity.this, "您选择了" + items[which], 0).show();
				//对话框消失
				dialog.dismiss();
			}
    	});
    	
    	builder.show();

###多选对话框
>设置标题

		AlertDialog.Builder builder = new Builder(this);
    	builder.setTitle("请选择你认为最帅的人");
>定义多选的选项，因为可以多选，所以需要一个boolean数组来记录哪些选项被选了

    	final String[] items = new String[]{
    			"Susion",
    			"SUSion",
    			"SUSion",
    			"SUSion"
    	};
		//true表示对应位置的选项被选了
    	final boolean[] checkedItems = new boolean[]{
    			true,
    			false,
    			false,
    			false,
    	};
    	builder.setMultiChoiceItems(items, checkedItems, new OnMultiChoiceClickListener() {

			//点击某个选项，如果该选项之前没被选择，那么此时isChecked的值为true
			@Override
			public void onClick(DialogInterface dialog, int which, boolean isChecked) {
				checkedItems[which] = isChecked;
			}
		});
    	
    	builder.setPositiveButton("确定", new OnClickListener() {
			
			@Override
			public void onClick(DialogInterface dialog, int which) {
				StringBuffer sb = new StringBuffer();
				for(int i = 0;i < items.length; i++){
					sb.append(checkedItems[i] ? items[i] + " " : "");
				}
				Toast.makeText(MainActivity.this, sb.toString(), 0).show();
			}
		});

    	builder.show();
