title: Andorid-自定义文件索引
date: 7/22/2016 2:30:48 PM  
categories: Android
---


#前言
- 一个检测任务文件夹下有许多以标签号为名称的文件夹
- 在对任务进行检测时必须要按照标签号的顺序一个一个的进行检测
- 但是标签号在检测任务文件夹下并一定是按照"自己的顺序"进行排列的
- 没有按照顺序排列，结果： 就是如果按照文件夹顺序一个一个的解析标签号，那么最终肯定不是按照顺序来的

##解决问题
>- 按照顺序解析 检测任务 下的 标签号文件
>- 解决方法： 按照文件的名字， 给标签号文件建索引， 即把顺序记录在集合中


	//建立索引所使用的集合
	private TreeMap<String, String> labelIndexSet = new TreeMap<>();    //1.帮标签号文件夹建立索引 2.用于根据标签号， 获得标签号文件夹的路径
    private List<String> labelIndexList = new ArrayList<>();

	//核心方法
 	CheckUtils.buildIndexForTaskLabel(checkTaskName, labelIndexSet, labelIndexList);


	// TreeMap<String, String> labelIndexSet 保证放入集合这种的标签号名字都是按照顺序排列的！！！
 	public static void buildIndexForTaskLabel(String taskName, TreeMap<String, String> labelIndex, List<String> labelIndexList) {

        File taskDir = new File(Constant.DATA_TASKS_DIR+File.separator+taskName);
        File[] files = taskDir.listFiles();

        for(File file : files){
            if(file.isDirectory()){
                if(!file.getName().equals(TASK_EXTRA_INFO_DIR)){
                    labelIndex.put(file.getName(), file.getAbsolutePath());   
                }
            }
        }

        Set<String> labelStrings = labelIndex.keySet();
        if(labelStrings != null){
            for(String label : labelStrings){
                labelIndexList.add(label);
            }
        }   //完毕后 labelIndexList 保存的有序的标签号集合
    }


- 经过上面的步骤
	- labelIndexList 保存的有序的标签号集合
	- labelIndexSet 用于得到标签号对应的其文件夹路径


#为何要这样做
- 这样做的主要原因是没有使用数据库对数据进行存储
- 如果使用数据库进行检测任务的存储，那么很容易就可以得到有序的标签号集合
- 好处
	- 只是稍微的构建了一下索引
	- 避免了数据操作的麻烦
	- 软件的整体运行速度变得更快

