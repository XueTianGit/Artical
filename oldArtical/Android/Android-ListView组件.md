title: 我也不知道标题该怎么写2
date: 2016/2/28 12:20:20  
categories: Android
---

# Android-ListView组件 #

它是Android中用于显示一行一行条目的组件，每一个条目都是一个View组件。它的应用十分广泛， 例如一些新闻客户端app的一条条新闻的展现
使用ListView能够很好的节省内存资源。
	
## 使用步骤 ##
1. 在布局文件中定义<ListView>组件， 对于ListView中的条目， 一般也使用一个单独的布局文件来定义。
2. 在Activity中控制ListView的显示

## 使用详解 ##

ListView就是一个条目的容器，一般条目中封装着我们想要显示的数据，我们应该把条目添加到ListView中。

1） 设置适配器 

通过设置各种适配器，我们可以在ListView中展现不同的数据。（一般数据都是存储在集合中的）。
	
	ListView. setAdapter(ListAdapter adapter) 
> ListAdapter是一个接口，通过查看手册知，这个接口并不需要我们来实现，我们可以继承其实现类。
> 它的常用的实现类有：ArrayAdapter  BaseAdapter SimpleAdapter等

> - 例如： 我们可以继承BaseAdapter抽象类，实现其getCount()和getView()方法。还有其他几个方法，我们首先得关注这两个。
> - 这两个方法都是由系统调用：
> - getCount(): 系统通过调用这个方法，得到要调用getView()方法的次数。
> - getView()： 系统把这个方法返回的View，作为ListView的条目。

范例：

>在这个例子的，把每个条目的数据都封装到集合中，由于getView（）返回的View就作为ListView的条目，因此getCount的返回值就是集合的大小。

> 对于getView（）的参数

	position代表：本次getView方法调用所返回的View对象，在listView中是处于第几个条目，那么position的值就是多少
	convertView: 与View对象的缓存相关
	parent： 条目的父亲

	class MyAdapter extends BaseAdapter{
		
		List<Person> itemList;

    	//系统调用，用来获知集合中有多少条元素
		@Override
		public int getCount() {
			return itemList.size();
		}
		
		//由系统调用，获取一个View对象，作为ListView的条目
		@Override
		public View getView(int position, View convertView, ViewGroup parent) {
			Person p = itemList.get(position);
			//将条目布局文件转成View，并填充数据，然后返回
		}
		
		@Override
		public Object getItem(int position) {
			return null;
		}

		@Override
		public long getItemId(int position) {
			return 0;
		}
    	
    }

- 将条目布局文件作为ListView的条目View

>- 从上面知道，ListView会把getView返回的View作为他的条目。因此在getView方法中，我们就要想办法把条目布局文件转换成View对象，
> - 并把数据封装到View中，以便显示。
> - 共有3种方法实现这个步骤,这三个方法原理都是一样的，都会使用LayoutInflater（布局填充器）。

	a:
	使用 ： public static View inflate (Context context, int resource, ViewGroup root) 
		-> View v = View.inflate(context, R.layout.item_lv, null);
	这个方法，会把一个xml布局文件填充成一个View对象。  
	 
	b:
	->先获取布局填充器   LayoutInflater inflater = LayoutInflater.from(context);
	->再进行填充        View v = inflater.inflate(R.layout.item_lv, null);
	
	c:
	->另一种方式获取布局填充器     LayoutInflater inflater = (LayoutInflater)context.getSystemService（Context.LAYOUT_INFLATER_SERVICE);
	->进行填充 。。。。。。


任选一种，这样我们就可以的到View对象，然后我们就可以根据position参数，获取集合中的数据，填充到View中，然后将View对象返回。


## ArrayAdapter ##

> 适用场景：
> 每个条目的数据都是同一种类型时。适用他非常方便。但扩展性差

例如：下面这个例子的条目只有一个数据（字符串）。

		String[] objects = new String[]{
				"霸天虎",
				"威震天",
				"大黄蜂"	
		};
		
		ListView lv = (ListView) findViewById(R.id.lv);
		//R.id.tv_name 是条目布局文件中，数据（就是本例中的字符串）填充位置的资源ID
		lv.setAdapter(new ArrayAdapter<String>(this, R.layout.item_listview, R.id.tv_name, objects));



## SimpleAdapter ##
	构造方法：
	SimpleAdapter(Context context, List<? extends Map<String, ?>> data, int resource, String[] from, int[] to) 

> 可以看出，这个适配器，要求条目中数据必须是这种形式  Map<String, ?
> 那么，这样扩展性就大了

范例：

		List<Map<String, Object>> data = new ArrayList<Map<String,Object>>();
		
		Map<String, Object> map1 = new HashMap<String, Object>();
		map1.put("photo", R.drawable.photo1);
		map1.put("name", "霸天虎");
		data.add(map1);
		
		Map<String, Object> map2 = new HashMap<String, Object>();
		map2.put("photo", R.drawable.photo2);
		map2.put("name", "威震天");
		data.add(map2);
		
		Map<String, Object> map3 = new HashMap<String, Object>();
		map3.put("photo", R.drawable.photo3);
		map3.put("name", "大黄蜂");
		data.add(map3);
		
		//会把map中封装的数据按名字填充到响应的资源ID的位置上
		//  photo -> R.id.iv_photo
		//  name  -> R.id.tv_name
		lv.setAdapter(new SimpleAdapter(this, data, R.layout.item_listview, 
				new String[]{"photo", "name"}, new int[]{R.id.iv_photo, R.id.tv_name}));


## ListView的优化 ##

### 优化一： 使用缓存的View对象 ###
> - 缺点的由来：
> - 由于每次屏幕的滑动，getView（）方法都会调用，布局文件都会被填充成View对象（这件事是非常耗费内存的）。

>- 解决：
> - public View getView(int position, View convertView, ViewGroup parent)
> - 对于converView参数，他就代表上次返回的那个View对象（这个对象是条目布局文件填充而产生的），
> - 因此，若每个条目的布局文件都相同，我们就完全没必要每次都填充布局文件了，
> 只需要直接把集合数据封装到这个缓存的View对象，然后返回他即可。

代码体现：

		if(convertView == null){
			//把布局文件填充成一个View对象
			v = View.inflate(MainActivity.this, R.layout.item_listview, null);
		}
		else{
			v = convertView;
		}

### 使用ViewHolder优化 ###
> - 缺点都由来：
> - 在我们每次往View对象封装数据时都要重复这个步骤：

	TextView tv_name = (TextView) v.findViewById(R.id.tv_name);
	tv_name.setText(p.getName());

虽然findViewById（）这个方法不是多么耗费资源，但毕竟经不起多次调用。但这一步，确实没必要每次都来一次，就像上面一样。

>- 解决思路：我们把ID记住不就OK了。
>- 步骤：
>- 定义一个类，条目的布局文件中有什么组件，这个类里就定义什么属性。这里这个类就叫ViewHolder吧。
>- 由于我们这个类的实例对象，是为了缓存组件的，因此我们每次都得在getView方法中得到这个实例对象，这里我们可以把ViewHolder对象
>- 利用View.setTag(mHolder);存到View中以便每次获得


代码实现：

	public View getView(int position, View convertView, ViewGroup parent) {

			News news = newsList.get(position);
			View v = null;
			ViewHolder mHolder;
			if(convertView == null){
				v = View.inflate(MainActivity.this, R.layout.item_listview, null);
				mHolder = new ViewHolder();
				//把布局文件中所有组件的对象封装至ViewHolder对象中
				mHolder.tv_title = (TextView) v.findViewById(R.id.tv_title);
				mHolder.tv_detail = (TextView) v.findViewById(R.id.tv_detail);
				mHolder.tv_comment = (TextView) v.findViewById(R.id.tv_comment);
				mHolder.siv = (SmartImageView) v.findViewById(R.id.iv);
				//把ViewHolder对象封装至View对象中
				v.setTag(mHolder);
			}
			else{
				v = convertView;
				mHolder = (ViewHolder) v.getTag();
			}
			//给三个文本框设置内容
			mHolder.tv_title.setText(news.getTitle());
			
			mHolder.tv_detail.setText(news.getDetail());
			
			mHolder.tv_comment.setText(news.getComment() + "条评论");
			
			//给新闻图片imageview设置内容
			mHolder.siv.setImageUrl(news.getImageUrl());
			return v;
		}
		
		class ViewHolder{
			//条目的布局文件中有什么组件，这里就定义什么属性
			TextView tv_title;
			TextView tv_detail;
			TextView tv_comment;
			SmartImageView siv;
		}
		



