

描述符特权级DPL、当前特权级CPL和请求特权级RPL的含义是什么？在哪些寄存器中这些字段？对应的访问条件是什么？ (challenge)写出一些简单的小程序（c or asm）来体现这些特权级的区别和联系。
 ```
  + 建议参见链接“ http://blog.csdn.net/better0332/article/details/3416749 ”对特权级的表述，并查阅指令手册。
  - 
 ```
   DPL存储在段描述符中，规定访问该段的权限级别(Descriptor Privilege Level)，每个段的DPL固定,代表真正的特权级。
   CPL是当前进程的权限级别(Current Privilege Level)，是当前正在执行的代码所在的段的特权级，存在于cs寄存器的低两位。
   RPL说明的是进程对段访问的请求权限(RequestPrivilegeLevel)，是对于段选择子而言的，每个段选择子有自己的RPL，它说明的是进程对段访问的请求权限，有点像函数参数。
   
    访问条件：
一、对数据段和堆栈段访问时的特权级控制：max {CPL, RPL} ≤ DPL
二、对代码段访问的特权级控制（代码执行权的特权转移）：
	（1） 普通转跳
  	 1.目标是一致代码段：要求CPL(CS.RPL)>=DestinationDescriptorCode.DPL
	 2.目标是非一致代码段： 要求CPL(CS.RPL)＝DestinationDescriptorCode.DPL　AND　RPL≤CPL(CS.RPL)
	（2）通过调用门的跳转
	 1.目标是一致代码段：要求CPL(CS.RPL)≥DestinationDescriptorCode.DPL ，RPL被清0
	 2.目标是非一致代码段：
		a.当用JMP指令跳转时： 要求CPL(CS.RPL)＝DestinationDescriptorCode.DPL　AND　RPL<= CPL(CS.RPL)
		b.当用CALL指令跳转时：要求CPL(CS.RPL)≥DestinationDescriptorCode.DPL（RPL被清0，不检查）,当CPL＞DPL时，程序跳转后CPL=DPL,特权级发生跃迁，这是我们当目前位置唯一见到的使程序当前执行忧先级(CPL)发生变化的跳转方法，即用CALL指令+调用门方式跳转，且目标代码段是非一致代码段。


比较不同特权级的中断切换时的堆栈变化差别；(challenge)写出一些简单的小程序（c or asm）来显示出不同特权级的的中断切换的堆栈变化情况。
- [x]  
   
  特权级变换，就会涉及到两个堆栈，外层堆栈（调用者堆栈）和内层堆栈（被调用者堆栈）。涉及到两个堆栈，那就得使用到TSS(Task-State Stack)取得其余堆栈的ss和esp。TSS里面包含多个字段，现在只关心偏移4到偏移27的3个ss和3个esp。我们有四个特权级ring0,ring1,ring2 & ring3,当转移是从外层到内层时（低特权级到高特权级），利用call指令，新的堆栈ss & esp才会从TSS中取得，因此只需记录ring0,ring1,ring2 的ss 和 esp 。而特权级从高到低转移时，必须用retf指令。
> 

 call
准备工作：
1.准备一个调用门描述符和调用门选择子（注意描述符的DPL大于等于源代码段的CPL和RPL），并初始化
2.准备TSS，只需要目标段DPL的堆栈位置即可
3.加载TSS
4.call调用门
 
 
B.retf
总结就四个压栈：
1.压入目标段的SS
2.压入目标段的esp
3.压入目标段的CS
4.压入目标段的eip
5.retf即可
