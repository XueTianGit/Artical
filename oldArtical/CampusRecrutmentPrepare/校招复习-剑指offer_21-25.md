title: 校招复习-剑指offer_21-25
date: 8/4/2016 9:47:25 AM       
categories: CampusRecrutmentPrepare
---


#解题代码
> 代码：

	
	/**
	 * Created by susion on 2016/7/31.
	 */
	public class ArrowToOffer05 {
	
	
	    @Test
	    public void test(){
	        TreeNode node10 = new TreeNode(10);
	        TreeNode node5 = new TreeNode(5);
	        TreeNode node12 = new TreeNode(12);
	        TreeNode node4 = new TreeNode(4);
	        TreeNode node7 = new TreeNode(7);
	
	        node10.left = node5;
	        node10.right  = node12;
	        node5.left = node4;
	        node5.right = node7;
	
	        FindPath(node10, 22);
	
	    }
	
	
	    /*
	    * 题目21描述:
	      输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否为该栈的弹出顺序。
	      假设压入栈的所有数字均不相等。例如序列1,2,3,4,5是某栈的压入顺序，序列4，5,3,2,1是该压栈序列对应的一个弹出序列，
	      但4,3,5,1,2就不可能是该压栈序列的弹出序列。（注意：这两个序列的长度是相等的）
	
	      思路:
	          1. 维护好当前栈顶元素， 如果弹出元素位于当前栈顶元素之前则失败  ()
	          2.
	            借用一个辅助的栈，遍历压栈顺序，先讲第一个放入栈中，这里是1，然后判断栈顶元素是不是出栈顺序的第一个元素，这里是4，很显然1≠4，所以我们继续压栈，直到相等以后开始出栈，出栈一个元素，则将出栈顺序向后移动一位，直到不相等，这样循环等压栈顺序遍历完成，如果辅助栈还不为空，说明弹出序列不是该栈的弹出顺序。
	            举例：
	            入栈1,2,3,4,5
	            出栈4,5,3,2,1
	            首先1入辅助栈，此时栈顶1≠4，继续入栈2
	            此时栈顶2≠4，继续入栈3
	            此时栈顶3≠4，继续入栈4
	            此时栈顶4＝4，出栈4，弹出序列向后一位，此时为5，,辅助栈里面是1,2,3
	            此时栈顶3≠5，继续入栈5
	            此时栈顶5=5，出栈5,弹出序列向后一位，此时为3，,辅助栈里面是1,2,3
	            ….
	            依次执行，最后辅助栈为空。如果不为空说明弹出序列不是该栈的弹出顺序。
	    */
	    public boolean IsPopOrder(int [] pushA,int [] popA) {
	
	        if(pushA.length == 0) return  false;
	
	        Stack<Integer> stack = new Stack<>();
	
	
	        for(int i=0, j=0; i<pushA.length; i++){
	
	            stack.push(pushA[i]);
	
	            while (j<popA.length && popA[j] == stack.peek()){
	                stack.pop();
	                j++;
	            }
	
	        }
	
	        return stack.empty();
	    }
	
	
	
	    /*
	    * 题目22描述:
	      从上往下打印出二叉树的每个节点，同层节点从左至右打印。
	      思路:
	        广度优先遍历, 利用队列实现
	    */
	
	    public ArrayList<Integer> PrintFromTopToBottom(TreeNode root) {
	
	        ArrayList<Integer> resultList = new ArrayList<>();
	        Queue<TreeNode> queue = new LinkedList<>();
	
	        if(root == null) return  resultList;
	        queue.add(root);
	
	        while(!queue.isEmpty()){
	
	            TreeNode treeNode = queue.remove();
	
	            if(treeNode.left != null){
	                queue.add(treeNode.left);
	            }
	
	            if(treeNode.right != null){
	                queue.add(treeNode.right);
	            }
	
	            resultList.add(treeNode.val);
	
	        }
	
	        return  resultList;
	    }
	
	
	
	    /*
	    * 题目23描述:
	      输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历的结果。
	      如果是则输出Yes,否则输出No。假设输入的数组的任意两个数字都互不相同。
	      思路:
	
	         由于是二叉搜索树，  则根据 父节点 的左子树的节点值肯定小于父节点， 而右子树的节点值肯定大于父节点
	
	         则，在数组中利用这个关系，判断出左右子数组也必须满足这个关系，否则不是二叉搜索树的后序遍历结果
	
	    */
	
	    public boolean VerifySquenceOfBST(int [] sequence) {
	
	        if(sequence.length == 0){
	            return false;
	        }
	
	        if(recusivVerifySequenceOfBST(sequence,  0, sequence.length - 2)){
	            System.out.print("Yes");
	            return true;
	        }else {
	            System.out.print("No");
	            return false;
	        }
	    }
	
	    private boolean recusivVerifySequenceOfBST(int[] sequence, int start, int end) {
	
	        if(end <= start){
	            return true;
	        }
	
	        int rootValue = sequence[end+1];
	
	        int leftEndIndex = end;
	        int i;
	        for(i=start; i<=end; i++){
	            if(sequence[i] > rootValue) {
	                leftEndIndex = i -1;
	                break;
	            }
	        }
	
	        for(; i<=end; i++){
	            if(sequence[i] < rootValue){
	                return  false;
	            }
	        }
	
	        if(recusivVerifySequenceOfBST(sequence,  start, leftEndIndex-1) &&
	                recusivVerifySequenceOfBST(sequence, leftEndIndex+1, end-2)){
	            return true;
	        }
	
	        return false;
	    }
	
	    /*
	        23解法， 另一种思路， 非递归版本
	    *左子树一定比右子树小，因此去掉根后，数字分为left，right两部分.
	    * right部分的最后一个数字是右子树的根他也比左子树所有值大，
	    * 因此我们可以每次只看有子树是否符合条件即可，
	    * 即使到达了左子树左子树也可以看出由左右子树组成的树还想右子树那样处理
	    *对于左子树回到了原问题，对于右子树，左子树的所有值都比右子树的根小可以暂时
	    * 把他看出右子树的左子树只需看看右子树的右子树是否符合要求即可
	    * */
	    public boolean VerifySquenceOfBST2(int [] sequence) {
	        int size = sequence.length;
	        if(0==size)return false;
	
	        int i = 0;
	        while(--size >= 0)
	        {
	            while(sequence[i++]<sequence[size]);
	            while(i<size && sequence[i++]>sequence[size]);
	
	            if(i<size)return false;
	            i=0;
	        }
	        return true;
	    }
	
	
	
	      /*
	       题目24描述:
	      输入一颗二叉树和一个整数，打印出二叉树中结点值的和为输入整数的所有路径。
	      路径定义为从树的根结点开始往下一直到叶结点所经过的结点形成一条路径。
	      思路:
	    */
	      public ArrayList<ArrayList<Integer>> FindPath(TreeNode root,int target) {
	
	          if(root == null) return null;
	
	          Stack<Integer> stack = new Stack<>();
	          ArrayList<ArrayList<Integer>> resultList = new ArrayList<>();
	          Queue<TreeNode> queue = new LinkedList<>();
	          int stackSum = 0;
	          int tempSum;
	
	          queue.add(root);
	
	          while(!queue.isEmpty()){
	
	              TreeNode treeNode = queue.remove();
	              tempSum = stackSum + treeNode.val;
	
	              if(tempSum < target){
	                  stack.add(treeNode.val);
	                  stackSum += treeNode.val;
	
	                  if(treeNode.left != null){
	                      queue.add(treeNode.left);
	                  }
	
	                  if(treeNode.right != null){
	                      queue.add(treeNode.right);
	                  }
	              }
	
	              if(tempSum == target){
	                  //打印出当前栈的集合，并把值弹出
	                  ArrayList<Integer> list = new ArrayList<>();
	                  for(int i=0; i<stack.size()-1; i++){
	                      list.add(stack.get(i));
	                  }
	
	                  list.add(treeNode.val);
	                  resultList.add(list);
	              }
	          }
	
	          return resultList;
	      }
	
	
	}







