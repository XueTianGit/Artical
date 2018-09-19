title: 校招复习-剑指offer_16-20
date: 8/4/2016 9:55:31 AM      
categories: CampusRecrutmentPrepare
---

#解题代码
> 代码：

	import java.util.ArrayList;
	import java.util.Stack;
	
	public class ArrowToOffer04 {
	
	
	    @Test
	    public void test(){
	
	        int[][] matriax = {{1, 2}, {3,4}};
	
	        printMatrix(matriax);
	    }
	
	    /*
	    * 题目16描述:
	      输入两个单调递增的链表，输出两个链表合成后的链表，当然我们需要合成后的链表满足单调不减规则。
	
	      思路:
	      比较节点值， 注意链接
	    */
	
	    public ListNode Merge(ListNode list1,ListNode list2) {
	
	
	        if(list1 == null && list2 == null){
	            return null;
	        }
	
	        if(list1 == null){
	            return  list2;
	        }
	
	        if(list2 == null){
	            return  list1;
	        }
	
	        ListNode p1 = list1;
	        ListNode p2 = list2;
	        ListNode head;
	        ListNode tail;
	
	        if(p1.val >= p2.val){
	            head = p2;
	            p2 = p2.next;
	        }else {
	            head = p1;
	            p1 = p1.next;
	        }
	
	        tail = head;
	
	        while(p1 != null && p2 != null){
	
	            if(p1.val >= p2.val){ // 问题出这里！！！！
	                tail.next = p2;
	                tail = p2;
	                p2 = p2.next;
	            }else {
	                tail.next = p1;
	                tail = p1;
	                p1 = p1.next;
	            }
	        }
	
	
	
	        if(p1 != null){
	            tail.next = p1;
	        }
	
	        if (p2 != null){
	            tail.next = p2;
	        }
	
	        return head;
	    }
	
	
	     /*
	    * 题目17描述:
	      输入两棵二叉树A，B，判断B是不是A的子结构。（ps：我们约定空树不是任意一个树的子结构）
	      思路:
	      "我们约定空树不是任意一个树的子结构"  ----> 只是在刚开始判断的时候起作用
	    */
	     public boolean HasSubtree(TreeNode root1,TreeNode root2) {
	
	         if(root1 == null || root2 == null) return false;
	
	         return DoesTree1HaveTree2(root1,root2) ||
	                 HasSubtree(root1.left, root2) ||HasSubtree(root1.right, root2);
	
	     }
	
	    public boolean DoesTree1HaveTree2(TreeNode root1,TreeNode root2){
	        if(root1 == null && root2 != null) return false;
	        if(root2 == null) return true;
	        if(root1.val != root2.val) return false;
	        return DoesTree1HaveTree2(root1.left, root2.left) && DoesTree1HaveTree2(root1.right, root2.right);
	    }
	
	
	    /*
	    * 题目18描述:
	      操作给定的二叉树，将其变换为源二叉树的镜像。
	      思路: 翻转每个节点的左右子节点
	
	            对于递归： 如果每条路是单一选择的， 那么就不要写出 if(){}  if(){} ... 这种判断！！！！！
	                    应使用单一选择路线！！！！！！
	    */
	
	    public void Mirror(TreeNode root) {
	
	        if( (root == null) || (root.left == null && root.right == null) ) return;
	
	        if(root.left == null && root.right != null){
	            root.left = root.right;
	            root.right = null;
	            Mirror(root.left);
	        }else if(root.right == null && root.left != null){
	            root.right = root.left;
	            root.left = null;
	            Mirror(root.right);
	        }else{
	
	            TreeNode temp = root.left;
	            root.left = root.right;
	            root.right = temp;
	
	            Mirror(root.left);
	            Mirror(root.right);
	        }
	
	    }
	
	
	    /*更简化......*/
	    public void Minoor2(TreeNode root){
	
	        if(root == null) return;
	
	        TreeNode temp = root.left;
	        root.left = root.right;
	        root.right = temp;
	
	        Mirror(root.left);
	        Mirror(root.right);
	    }
	
	    /*什么数据结构并不重要， 关键是要实现对树中的每个节点实现遍历按， 并交换左右子节点*/
	    public void Minoor3(TreeNode root){
	
	        if(root == null) return;
	
	        Stack<TreeNode> stackNode = new Stack<>();
	        stackNode.push(root);
	
	        while (!stackNode.empty()){
	            TreeNode tree = stackNode.pop();
	
	            if(tree.left != null || tree.right != null){
	                TreeNode temp = tree.left;
	                tree.left = tree.right;
	                tree.right = temp;
	            }
	
	            if(tree.left != null){
	                stackNode.push(tree.left);
	            }
	
	            if(tree.right != null){
	                stackNode.push(tree.right);
	            }
	        }
	
	    }
	
	
	  /*
	    * 题目19描述:
	      输入一个矩阵，按照从外向里以顺时针的顺序依次打印出每一个数字，
	      例如，如果输入如下矩阵： 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 则依次打印出数字1,2,3,4,8,12,16,15,14,13,9,5,6,7,11,10.
	      思路:
	    */
	    public ArrayList<Integer> printMatrix(int [][] matrix) {
	
	        ArrayList<Integer> resultList = new ArrayList<>();
	
	        int colNumber = matrix[0].length;
	        int rowNumber = matrix.length;
	
	        int col = 0; int row = 0;
	        int circleNumber = 0;
	        int n = ((colNumber <= rowNumber ? colNumber : rowNumber) - 1) / 2;
	
	        while (circleNumber <= n ){
	
	            for(; col<colNumber-circleNumber; col++){
	                resultList.add(matrix[row][col]);
	            }
	
	            for(col--,row++; row<rowNumber-circleNumber; row++){
	                resultList.add(matrix[row][col]);
	            }
	
	            circleNumber++;
	
	            for(row--,col--; (col >= circleNumber) && (circleNumber-1 <= n); col--){
	                resultList.add(matrix[row][col]);
	            }
	
	            for(row--, col++; row >= circleNumber && circleNumber-1 <= n; row--){
	                resultList.add(matrix[row][col]);
	            }
	
	
	            row = col = circleNumber;
	        }
	
	        return  resultList;
	    }
	
	
	
	    /*
	      题目20描述:
	      定义栈的数据结构，请在该类型中实现一个能够得到栈最小元素的min函数
	    *  思路： 感觉这样实现 是不是有些扯淡。。。。。(其实我没读懂题目)
	    * */
	  /*
	    private List<Integer> mStack = new ArrayList<>();
	
	    public void push(int node) {
	       mStack.add(0, node);
	    }
	
	    public void pop() {
	        mStack.remove(0);
	    }
	
	    public int top() {
	        return mStack.get(0);
	    }
	
	    public int min() {
	        return Collections.min(mStack);
	    }
	
	   */
	    /*20 题目好像是这个意思：      定义栈的数据结构，请在该类型中实现一个能够得到栈最小元素的min函数， 时间复杂度为 o(1)*/
	
	
	    private Stack<Integer> mStack = new Stack<>();
	    private Stack<Integer> minStack = new Stack();
	    private int currentMinNumber = 0;
	
	    public void push(int node) {
	
	        if(minStack.empty()){
	            minStack.push(node);
	            currentMinNumber = node;
	            mStack.push(node);
	        }else{
	
	            mStack.push(node);
	
	            if(node <= currentMinNumber){
	                currentMinNumber = node;
	                minStack.push(node);
	            }
	
	        }
	
	    }
	
	    public void pop() {
	
	        Integer popNumber = mStack.pop();
	
	        if(popNumber == minStack.peek()){
	            minStack.pop();
	        }
	
	    }
	
	    public int top() {
	        return mStack.peek();
	    }
	
	    public int min() {
	        return minStack.peek();
	    }
	}