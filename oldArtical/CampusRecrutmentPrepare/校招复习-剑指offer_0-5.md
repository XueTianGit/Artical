title: 校招复习-剑指offer_0-5
date: 7/25/2016 8:16:51 PM    
categories: CampusRecrutmentPrepare
---


#解题代码
> 代码：


	/**
	 * Created by susion on 2016/7/25.
	 */
	public class ArrowToOffer01 {


	    @Test
	    public void test(){
	        int[] pre = {1,2,3,4};
	        int[] in = {1,2,3,4};
	        reConstructBinaryTree(pre, in);
	
	    }
	
	    /*
	      题目1
	    * 查找： 方向要明确： 从左下角开始查找：向上数字递减， 向右数字递增  (很明确)   ----> 右上角也是可以的
	    *
	    * */
	    public boolean findIn2DimensionArray(int [][] array,int target) {
	
	        if(array == null){
	            return false;
	        }
	
	        int startRow = array.length - 1;
	        int startCol = 0;
	
	        while(startRow >= 0 && startCol <= array[0].length - 1){
	
	            if(array[startRow][startCol] == target){
	                return true;
	            }
	
	            if(array[startRow][startCol] > target){
	                startRow--;
	                continue;
	            }
	
	
	            if(array[startRow][startCol] < target){
	                startCol++;
	                continue;
	            }
	
	        }
	        return false;
	    }
	
	
	    /**
	     * 对每一行进行二分查找
	     *
	     */
	    public boolean findIn2DimensionArrayMethod2(int [][] array,int target) {
	
	        if(array == null){
	            return false;
	        }
	
	        for(int i=0; i< array.length; i++){
	            if(binarySearchInIntArray(array[i], target)){
	                return true;
	            }
	        }
	        return false;
	    }
	
	    private boolean binarySearchInIntArray(int[] array, int target) {
	
	        int low = 0;
	        int high = array.length - 1;
	
	        if(low <= high){
	
	            int mid = (low + high) / 2;
	
	            if(target > array[mid]){
	                low = mid + 1;
	            }
	
	            if(target < array[mid]){
	                high = mid - 1;
	            }
	
	            if(target == array[mid]){
	                return  true;
	            }
	        }
	
	        return false;
	    }
	
	
	    /*
	      题目2
	    * 替换字符串中的空格
	    * 题目： 请实现一个函数，将一个字符串中的空格替换成“%20”。例如，当字符串为We Are Happy.则经过替换之后的字符串为We%20Are%20Happy。
	    *
	    * 从后向前插入可以减少移动次数
	    * */
	    public String replaceSpace(StringBuffer str) {
	
	        String s = str.toString();
	        int blackNumber = 0;
	        for(int i=0; i<s.length(); i++){
	            if (s.charAt(i) == ' ') {
	                blackNumber++;
	            }
	        }
	
	        int newLength = s.length() + blackNumber * 2;
	        char[] newString  = new char[newLength];
	        int progress = newLength - 1;
	
	        for(int i=s.length()-1; i>=0; i--){
	            if(s.charAt(i) == ' '){
	                newString[progress--] = '0';
	                newString[progress--] = '2';
	                newString[progress--] = '%';
	            }else {
	                newString[progress] = s.charAt(i);
	                progress--;
	            }
	        }
	
	        return new String(newString);
	    }
	
	    /**
	     *题目3: 输入一个链表，从尾到头打印链表每个节点的值
	     * 思路一： 递归
	     * 思路二： 利用栈
	     */
	    public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
	        ArrayList<Integer> resultList  = new ArrayList<>();
	        recurisiveReversePrintList(listNode, resultList);
	        return resultList;
	    }
	
	    private void recurisiveReversePrintList(ListNode listNode, ArrayList<Integer> resultList) {
	        if(listNode != null){
	            if(listNode.next != null){
	                recurisiveReversePrintList(listNode.next, resultList);
	            }
	            resultList.add(listNode.val);
	        }
	    }
	
	
	
	    /**
	     *题目4: 输入某二叉树的前序遍历和中序遍历的结果，请重建出该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。
	     *       例如输入前序遍历序列{1,2,4,7,3,5,6,8}和中序遍历序列{4,7,2,1,5,3,8,6}，则重建二叉树并返回。
	     *
	     *思路： 1. 根据前序遍历分辨出左右子树
	     *      2. 将中序遍历的 “父节点” 插入为父节点
	     */
	
	    public TreeNode reConstructBinaryTree(int [] pre,int [] in) {
	
	        TreeNode resultTree = new TreeNode(pre[0]);
	
	        int preStartIndex = 0;
	        int preEndIndex = pre.length - 1;
	        int inStartIndex = 0;
	        int inEndIndex = in.length - 1;
	
	        recurisiveReConstructBinaryTree(resultTree, pre, in, preStartIndex, preEndIndex, inStartIndex, inEndIndex);
	
	        return resultTree;
	    }
	
	    private void recurisiveReConstructBinaryTree(TreeNode resultTree, int[] pre, int[] in, int preStartIndex, int preEndIndex, int inStartIndex, int inEndIndex) {
	
	        int parentVaule = resultTree.val;
	        int parentIndex = 0;
	        for(int i=inStartIndex; i<=inEndIndex;  i++){
	            if(in[i] == parentVaule){
	                parentIndex = i;
	                break;
	            }
	        }
	
	        int leftTreeNodeNumber = parentIndex - inStartIndex;
	        int rightTreeNodeNumber = inEndIndex - parentIndex;
	
	        if(leftTreeNodeNumber > 0){
	            //重建左子树
	            int leftPreEndIndex = preStartIndex + leftTreeNodeNumber;
	            int leftPreStartIndex = preStartIndex+1;
	            int leftInStartIndex = inStartIndex;
	            int leftInEndIndex = parentIndex - 1;
	
	            int leftValue = pre[leftPreStartIndex];
	            resultTree.left = new TreeNode(leftValue);
	            recurisiveReConstructBinaryTree(resultTree.left, pre, in, leftPreStartIndex, leftPreEndIndex, leftInStartIndex, leftInEndIndex);
	        }
	
	        if(rightTreeNodeNumber > 0){
	            //重建右子树
	
	            int rightPreStartIndex = preStartIndex + leftTreeNodeNumber + 1;
	            int rightPreEndIndex = rightPreStartIndex + rightTreeNodeNumber - 1;
	            int rightInStartIndex  = parentIndex + 1;
	            int rightInEndIndex = parentIndex + rightTreeNodeNumber;
	
	            int rightValue = pre[rightPreStartIndex];
	            resultTree.right = new TreeNode(rightValue);
	            recurisiveReConstructBinaryTree(resultTree.right, pre, in, rightPreStartIndex, rightPreEndIndex, rightInStartIndex, rightInEndIndex);
	        }
	
	    }
	
	
	
	
	    /**
	     *题目5: 用两个栈来实现一个队列，完成队列的Push和Pop操作。 队列中的元素为int类型。
	     *
	     * push: 将push的数压入栈1
	     * pop： 将栈1中的数都弹到栈2中， 将栈2中的栈顶数字取出
	     *
	     */
	
	    Stack<Integer> stack1 = new Stack<Integer>();
	    Stack<Integer> stack2 = new Stack<Integer>();
	
	    public void push(int node) {
	        stack1.push(node);
	    }
	
	    public int pop() {
	        if(stack2.empty()){
	            //将栈1的数弹到栈2中
	            while (!stack1.empty()){
	                stack2.push(stack1.pop());
	            }
	        }
	
	        return stack2.pop();
	    }
	
	}
	
	
	
	/*题目相关的数据结构*/
	class ListNode {
	        int val;
	        ListNode next = null;
	        ListNode(int val) {
	            this.val = val;
	        }
	}
	
	class TreeNode {
	    int val;
	    TreeNode left;
	    TreeNode right;
	    TreeNode(int x) { val = x; }
	}



