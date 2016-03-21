#lec9 虚存置换算法spoc练习

## 个人思考题
1. 置换算法的功能？

2. 全局和局部置换算法的不同？

3. 最优算法、先进先出算法和LRU算法的思路？

4. 时钟置换算法的思路？

5. LFU算法的思路？

6. 什么是Belady现象？

7. 几种局部置换算法的相关性：什么地方是相似的？什么地方是不同的？为什么有这种相似或不同？

8. 什么是工作集？

9. 什么是常驻集？

10. 工作集算法的思路？

11. 缺页率算法的思路？

12. 什么是虚拟内存管理的抖动现象？

13. 操作系统负载控制的最佳状态是什么状态？

## 小组思考题目

----
(1)（spoc）请证明为何LRU算法不会出现belady现象


(2)（spoc）根据你的`学号 mod 4`的结果值，确定选择四种替换算法（0：LRU置换算法，1:改进的clock 页置换算法，2：工作集页置换算法，3：缺页率置换算法）中的一种来设计一个应用程序（可基于python, ruby, C, C++，LISP等）模拟实现，并给出测试。请参考如python代码或独自实现。
 - [页置换算法实现的参考实例](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab3/page-replacement-policy.py)
 - 
 
---
 #include <iostream>
  #include <stdlib.h>
  #include <stdio.h>
  using namespace std;
  struct node 
  {
      int page_number;
  node * next;
  node * prev;
  };
  node * root;
  int T = 2;
  int n = 10;
  int page_visit[10] = {2,2,3,1,2,4,2,4,0,3};
  int t_last = -1 ;
  int t_current;
  int cache[5] = {0,3,4,-1,-1};
  int workset[2] = {0,3};
  bool page_miss(int x)
  {
  node * p = root;
  while(p != NULL)
  {
      if( p -> page_number == x)
      return 0;
      p = p -> next;
  }
  return 1;
  }
  bool not_in_workset(int x)
  {
  if(x != workset[0] && x != workset[1])
      return 1;
  return 0;
  }
  void func()
  {
  int size = 3;
  for(int i = 0;i<n;i++)
  {

      int x = page_visit[i];
      cout << "visiting: "<<char(x+97)<<endl;
      if(page_miss(x))  //page missing
      {
          t_current = i;
          if(t_current - t_last > T) //standing set is too big;
          {
                  cout <<"missing and the standing set is too big\n";
                  node * p = root;
                  node * p2 ;
                  while(p!=NULL)
                  {
                  //cout << workset[0]<< " "<<workset[1] <<endl;
                  //cout << not_in_workset(p -> page_number)<<endl;
                      if( not_in_workset(p -> page_number))
                      {
                          if( p != root)
                          {
                              if(p -> next == NULL)
                              {
                                  cout << "delete: "<< char(p -> page_number  + 97) <<endl;
                                  size --;
                                  p -> prev -> next = NULL;
                                  p -> prev = NULL;
                              }
                              else
                              {
                                  p -> prev -> next = p -> next;
                                  p -> next -> prev = p -> prev;
                                  cout << "delete: "<< char(p -> page_number  + 97) <<endl;
                                  size --;
                              }
                          }
                          if(p == root)
                          {
                              cout << "delete: "<< char(p -> page_number  + 97) <<endl;
                              size --;
                              root = p -> next;
                              root -> prev = NULL;
                          }
                      }
                      p2 = p;
                      p = p->next;

                  }

              node * q = new node();
              q -> page_number = x;
              cout <<"add: "<<char(x + 97)<<endl;
              p2 -> next = q;
              q -> prev = p2;
              q -> next = NULL;
          }
          else  //standing set is too small
          {
              cout <<"missing and the standing set is too small\n";
               cout <<"add: "<<char(x + 97)<<endl;
              node * q = new node();
              q -> page_number = x;
              node * p = root;
              while( p-> next != NULL)
                  p = p -> next;
              p -> next = q;
              q -> prev = p;
              q -> next = NULL;
          }
      }
  workset[0] = workset[1];
  workset[1] = x;



  }
  }
  int main()
  {
  root = new node();
  root -> prev = NULL;
  root -> page_number = 0;
  root -> next = new node();
  root -> next -> page_number = 3;
  node * p = root -> next;
  p -> prev = root;
  p -> next = new node();
  p -> next -> page_number = 4;
  p -> next -> prev = p;
  p -> next -> next = NULL;
  func();
  }
  ---
 
## 扩展思考题
（1）了解LIRS页置换算法的设计思路，尝试用高级语言实现其基本思路。此算法是江松博士（导师：张晓东博士）设计完成的，非常不错！

参考信息：

 - [LIRS conf paper](http://www.ece.eng.wayne.edu/~sjiang/pubs/papers/jiang02_LIRS.pdf)
 - [LIRS journal paper](http://www.ece.eng.wayne.edu/~sjiang/pubs/papers/jiang05_LIRS.pdf)
 - [LIRS-replacement ppt1](http://dragonstar.ict.ac.cn/course_09/XD_Zhang/(6)-LIRS-replacement.pdf)
 - [LIRS-replacement ppt2](http://www.ece.eng.wayne.edu/~sjiang/Projects/LIRS/sig02.ppt)
