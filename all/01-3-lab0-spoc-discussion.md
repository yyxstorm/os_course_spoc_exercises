



分析和实验funcall.c，需要完成的内容包括： 
-[X]

 - 修改代码，可正常显示小组两位同学的学号（用字符串） 
 - 生成funcall.c的汇编码，理解其实现并给汇编码写注释
 - 尝试用xem的简单调试功能单步调试代码
 - 回答如下问题：
   - funcall中的堆栈有多大？是内核态堆栈还是用户态堆栈                 112   内核态
   - funcall中的全局变量ret放在内存中何处？如何对它寻址？              放在堆中  利用sp+24寻址。
   - funcall中的字符串放在内存中何处？如何对它寻址？                     sp
   - 局部变量i在内存中的何处？如何对它寻址？                             sp+4
   - 当前系统是处于中断使能状态吗？                                      是
   - funcall中的函数参数是如何传递的？函数返回值是如何传递的？          参数列表从右至左压入栈    返回值放入a中
   - 分析并说明funcall执行文件的格式和内容
　


 
