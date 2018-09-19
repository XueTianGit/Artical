title: 校招复习-剑指offer_6-10
date: 7/26/2016 11:20:41 AM     
categories: CampusRecrutmentPrepare
---


#解题代码
> 代码：


	/**
	 * Created by susion on 2016/7/26.
	 */
	public class ArrowToOffer02 {
	
	    @Test
	    public void test(){
	        int[] a = {3,4,5,6,0,0,2};
	        System.out.print(Fibonacci(5));
	    }
	
	
	    /*
	    * 题目6描述:
	        把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。
	        输入一个递增排序的数组的一个旋转，输出旋转数组的最小元素。
	        例如数组{3,4,5,1,2}为{1,2,3,4,5}的一个旋转，该数组的最小值为1。
	        NOTE：给出的所有元素都大于0，若数组大小为0，请返回0。
	      思路： 分别从第一个元素和最后一个元素开始遍历
	            从第一个元素开始往后 子序列应该递增
	            从最后一个元素开始往前，子序列应该递减
	            一旦违反此规定，则到达分歧点，即找到最小元素
	    * */
	
	    public int minNumberInRotateArray(int [] array) {
	
	        if(array.length == 0){
	            return 0;
	        }
	
	        for(int i=0, j=array.length -1; i<array.length-1 && j > 0; i++, j--){
	
	            if(array[i] > array[i+1]){
	                return array[i+1];
	            }
	
	            if(array[j] < array[j-1]){
	                return array[j];
	            }
	        }
	        return 0;
	    }
	
	    /*
	    * 题目7描述:
	      大家都知道斐波那契数列，现在要求输入一个整数n，请你输出斐波那契数列的第n项。n<=39
	
	      思路: 保存计算结果， 不要做多余的计算
	    * */
	
	    public int Fibonacci(int n) {
	
	        if(n == 0){
	            return  0;
	        }
	
	        if(n == 1 || n == 2){
	            return  1;
	        }
	
	        int a = 1;  //代表 a_n-1
	        int b = 1;  //代表 a_n-2
	        int temp = 0;
	
	        int i = 3;
	        while( i < n){
	            temp = a;
	            a = b;
	            b = temp + b;
	            i++;
	        }
	
	        return a + b;
	    }
	
	    int FibonacciMethod2(int n) {
	        int f = 0, g = 1;
	        while(n-- > 0) {
	            g += f;
	            f = g - f;
	        }
	        return f;
	    }
	
	
	    /*
	    * 题目8描述:
	      一只青蛙一次可以跳上1级台阶，也可以跳上2级。求该青蛙跳上一个n级的台阶总共有多少种跳法。
	
	      思路:对于第n个台阶来说，只能从n-1或者n-2的台阶跳上来，所以
	            F(n) = F(n-1) + F(n-2)
	            斐波拉契数序列，初始条件
	            n=1:只能一种方法
	            n=2:两种
	    * */
	    public int JumpFloor(int target) {
	        int f = 1, g = 1;
	        while(target-- > 0) {
	            g += f;
	            f = g - f;
	        }
	        return f;
	    }
	
	
	    /*
	    * 题目9描述:
	      一只青蛙一次可以跳上1级台阶，也可以跳上2级……它也可以跳上n级。求该青蛙跳上一个n级的台阶总共有多少种跳法。
	      思路:
	       F(n) = 1 + F(n-1) + F(n-2) +...+f(1)
	    * */
	
	    public int JumpFloorII(int n) {
	
	        if(n == 0){
	            return  0;
	        }
	
	        int[] numbers = new int[n-1];   //numbers_i 代表此位置共有多少种跳法
	        int preSum = 0;                 //前面跳法的累积和
	
	        for(int i=0; i<numbers.length; i++){
	            numbers[i] = 1 + preSum;
	            preSum += numbers[i];
	        }
	
	        return preSum + 1;
	    }
	
	
	    //另一种解法
	    //思路： 每个台阶都有跳与不跳两种情况（除了最后一个台阶），最后一个台阶必须跳。所以共用2^(n-1)中情况
	    public int JumpFloorII2(int number) {
	        return  1<<--number;
	    }
	
	
	    /*
	    * 题目10描述:
	      我们可以用2*1的小矩形横着或者竖着去覆盖更大的矩形。请问用n个2*1的小矩形无重叠地覆盖一个2*n的大矩形，总共有多少种方法？
	      思路:
	        问题的根本依旧是 斐波那契数列
	            当 target = 0， 或 1 时；  返回1
	            当 target = 2时； 返回 2
	            当 target = n 分为两步考虑：
	                            第一次摆放一块 2*1 的小矩阵，则摆放方法总共为f(target - 1)
	                            第一次摆放一块1*2的小矩阵，则摆放方法总共为f(target-2)
	                                因为，摆放了一块1*2的小矩阵（用√√表示），对应下方的1*2（用××表示）摆放方法就确定了，所以为f(targte-2)
	    * */
	    public int RectCover(int target) {
	
	        if(target == 0){
	            return  0;
	        }
	
	        int f = 0, g = 1;
	        while(target-- > 0) {
	            g += f;
	            f = g - f;
	        }
	        return g;
	    }
	
	
	}



