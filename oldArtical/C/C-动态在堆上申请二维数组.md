title: C-动态在堆上申请二维数组
date: 2016/3/14 11:10:15      
categories: C
---

# C-动态在堆上申请二维数组 #

> 想要成功的动态在堆上申请二维数组成功，应真的明白二维数组的机构
> 
> - 二维数组的每一维都是一个一维数组
> - 对于 int a[3][3],  那么代表一维数组的常量指针分别为： a[0], a[1], a[2]
> - 二维数组的大小: 一维数 * 二维数

下面手工分配二维数组: a[2][2]:

	#include <stdio.h> 
	#include <string.h> 
	#include <process.h> 
	
	int** malloc2d(int row, int col)
	{
		int** ret = (int**)malloc(sizeof(int*) * row);  //指向二维数组，一维数组指针的二级指针 
		int* p = (int*)malloc(sizeof(int) * row * col);   //为二维数组的每个元素分配空间
		
		//分别为一维数组指针指向所拥有的二维数组元素的控件 
		int i = 0; 
		if( ret && p) //安全检查 
		{
			for(i=0; i<row; i++)
			{
				ret[i] = (p + i * col);  //ret[i] 是二维数组的一维数组指针  ret[i]  = *(ret + i) 
			}
		}
		else
		{
			free(ret);
			free(p);
			ret = NULL;
			p = NULL;
		}
		
		return ret;
	}
	
	void free2d(int** a)
	{
		free(a[0]); //对应p   必须先释放他！！ 
		free(a);   //对应ret 
	}
	
	
	int main()
	{
		int** a = malloc2d(2, 2);
		int i = 0;
		int j=0;
		
		for(i=0; i<2; i++)
		{
			for(j=0; j<2; j++)
			{
				a[i][j] = i + j;		
			}	
		}
		
		for(i=0; i<2; i++)
		{
			for(j=0; j<2; j++)
			{
				printf("a[%d][%d] = %d, ", i, j, *(*(a+i) + j));		
			}	
			printf("\n");
		}
		
		free2d(a);
	
		return 0;
	}


![](http://7xrbxa.com1.z0.glb.clouddn.com/C%E5%8A%A8%E6%80%81%E5%9C%A8%E5%A0%86%E4%B8%8A%E7%94%B3%E8%AF%B7%E4%BA%8C%E7%BB%B4%E6%95%B0%E7%BB%84.png)
 