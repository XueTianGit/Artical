title: 校招复习-剑指offer_11-15
date: 7/27/2016 4:54:37 PM      
categories: CampusRecrutmentPrepare
---


#解题代码
> 代码：

	/**
	 * Created by susion on 2016/7/26.
	 */
	public class ArrowToOffer03 {
	
	
	    @Test
	    public void test(){
	
	    }
	    /*
	    * 题目11描述:
	      输入一个整数，输出该数二进制表示中1的个数。其中负数用补码表示
	
	      思路: 把一个整数减去1，再和原整数做与运算，会把该整数最右边一个1变成0.那么一个整数的二进制有多少个1，就可以进行多少次这样的操作
	      举个例子：一个二进制数1100，从右边数起第三位是处于最右边的一个1。减去1后，第三位变成0，它后面的两位0变成了1，而前面的1保持不变，
	      因此得到的结果是1011.我们发现减1的结果是把最右边的一个1开始的所有位都取反了。这个时候如果我们再把原来的整数和减去1之后的结果做与运算，
	      从原来整数最右边一个1那一位开始所有位都会变成0。如1100&1011=1000.也就是说，。
	    * */
	    public int NumberOf1(int n) {
	        int count = 0;
	        while(n!= 0){
	            count++;
	            n = n & (n - 1);
	        }
	        return count;
	    }
	
	     /*
	    * 题目12描述:
	      给定一个double类型的浮点数base和int类型的整数exponent。求base的exponent次方。
	
	      思路:  1. 注意特殊值
	            2.  对于指数是负数的处理
	    * */
	
	    public double Power(double base, int exponent) {
	
	        if(exponent == 0){
	            return 1;
	        }
	        if( base > -0.0000001 && base < 0.0000001){
	            return  0;
	        }
	
	
	        if(exponent < 0){
	            return  1 / powNumber(base, -exponent);
	        }
	
	        return powNumber(base, exponent);
	    }
	
	    private double powNumber(double base, int exponent) {
	        double result = base;
	        for(int i=1; i<exponent; i++){
	            result *= base;
	        }
	        return  result;
	    }
	
	
	    /*
	    * 题目13描述:
	      输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有的奇数位于数组的前半部分，
	      所有的偶数位于位于数组的后半部分，并保证奇数和奇数，偶数和偶数之间的相对位置不变。
	      思路1:
	        空间换时间， 另外开辟o(n)的空间的区域，分成两个数组：奇数组和偶数组
	        将数组中的数划分到这两个区域中，然后再合并到原数组中
	      思路2：
	        类似冒泡排序， (冒泡排序是稳定的， 但时间复杂度比较高)
	            从第一个位置开始遍历， 当发现奇数时，插入前面的第一个偶数的前面
	                                 如果是偶数则继续移动
	    * */
	    public void reOrderArray(int [] array) {
	
	        //统计数组中奇数与偶数的分别数量
	        int oddNumber = 0;
	        int evenNumber = 0;
	        for(int i=0; i<array.length; i++){
	            if(array[i] % 2 == 0){
	                evenNumber++;
	            }else{
	                oddNumber++;
	            }
	        }
	
	        int[] oddArray = new int[oddNumber];
	        int[] evenArray = new int[evenNumber];
	
	        int oddIndex = 0;
	        int evenIndex = 0;
	        for(int i=0; i<array.length; i++){
	            if(array[i] % 2 != 0){
	                oddArray[oddIndex] = array[i];
	                oddIndex++;
	            }else{
	                evenArray[evenIndex] = array[i];
	                evenIndex++;
	            }
	        }
	
	        for(int i=0; i<oddArray.length; i++){
	            array[i] = oddArray[i];
	        }
	
	        for(int j=oddArray.length; j<array.length; j++){
	            array[j] = evenArray[j - oddArray.length];
	        }
	
	    }
	
	    public void reOrderArray2(int[] array){
	
	        boolean doExchange = true;
	
	        for(int i=1; i<array.length; i++){
	
	            if(array[i] % 2 == 1){
	                for(int j=i; j>0; j--){
	                    if (array[j - 1] % 2 == 0){
	                        int t = array[j];
	                        array[j] = array[j - 1];
	                        array[j - 1] = t;
	                    }
	                }
	            }
	        }
	
	    }
	
	      /*
	    * 题目14 描述:
	      输入一个链表，输出该链表中倒数第k个结点。
	      思路: 两个遍历指针： P1 P2，  两者相距K个节点， 当p2指到链表末尾时， p1的下一个节点即为所求节点
	
	      note: 要关注代码的健壮性！！！！！！
	    * */
	    public ListNode FindKthToTail(ListNode head,int k) {
	
	        if(head == null || k <= 0){
	            return  null;
	        }
	
	        ListNode p1 = head;
	        ListNode p2 = head;
	
	
	        for(int i=1; i<k; i++){
	            p1 = p1.next;
	            if(p1 == null){
	                return null;
	            }
	        }
	
	        while (p1.next != null){
	            p1 = p1.next;
	            p2 = p2.next;
	        }
	        return  p2;
	    }
	
	    /*
	    题目15描述:
	    输入一个链表，反转链表后，输出链表的所有元素。
	    思路:
	
	    */
	    public ListNode ReverseList(ListNode head) {
	
	        if(head == null){
	            return  null;
	        }
	
	        if(head.next == null){
	            return  head;
	        }
	
	        ListNode p2 = head.next;
	        ListNode p3 = p2.next;
	        ListNode temp;
	
	        p2.next = head;
	        head.next = null;  //必须要有, 不然会构成死循环
	        while (p3 != null){
	
	            temp = p3;
	            p3 = p3.next;
	
	            temp.next = p2;
	            p2 = temp;
	
	        }
	
	        return p2;
	    }
	}



