# 3.5 ucore系统调用分析
 1. ucore的系统调用中参数传递代码分析。
  - 答：在用户态中通过函数调用将参数传递至用户态的```syscall```函数（```syscall```参数长度可变）中，用户态的```syscall```函数通过```asm volatile```机制将参数传入内核态；在内核态中系统调用通过```trapframe(tf)```进行参数传递，最后在内核态的```syscall```函数中执行：
 
    ```
    int num = tf->tf_regs.reg_eax;
    if (num >= 0 && num < NUM_SYSCALLS) {
        if (syscalls[num] != NULL) {
            arg[0] = tf->tf_regs.reg_edx;
            arg[1] = tf->tf_regs.reg_ecx;
            arg[2] = tf->tf_regs.reg_ebx;
            arg[3] = tf->tf_regs.reg_edi;
            arg[4] = tf->tf_regs.reg_esi;
            tf->tf_regs.reg_eax = syscalls[num](arg);
            return ;
        }
    }
    ```

   其中系统调用类型码（用于判断使用哪个系统调用）通过```tf->tf_regs.reg_eax```进行传递，其他参数通过```tf->tf_regs```进行传递，系统调用的结果储存在```tf->tf_regs.reg_eax```中。 最后在用户态的```syscall(int,...)```函数中通过```eax```寄存器返回。

 2. 以getpid为例，分析ucore的系统调用中返回结果的传递代码。
   - 答：在内核态中，系统调用的返回值通过```trapframe```中的```tf_regs.reg_eax```进行传递，如下述代码所示：
 
     ```tf->tf_regs.reg_eax = syscalls[num](arg);```

     最后在用户态的syscall函数中通过```eax```寄存器返回。

 3. 以ucore lab8的answer为例，分析ucore 应用的系统调用编写和含义。
   - 答：以```getpid()```函数为例：
     - a. ```getpid()```函数是```ulib.c```中经过包装的系统调用；
     - b. ```getpid()```调用用户态```syscall.c```中的```sys_getpid()```函数；
     - c. ```sys_getpid()```调用用户态的```syscall(int,...)```函数；
     - d. 用户态的```syscall(int,...)```函数通过```asm volatile```机制将参数传入内核态；
     - e. 在内核态中参数（包括系统调用类型码）通过```trapframe```进行传递，最后传递到内核态的```syscall（）```函数中；
     - f. 内核态的```syscall()```函数根据系统调用类型调用```sys_getpid(int)```函数，返回值写入```eax```寄存器中；
     - g. 返回到用户态时，返回值从```eax```寄存器中读出。

 4. 以ucore lab8的answer为例，尝试修改并运行ucore OS kernel代码，使其具有类似Linux应用工具`strace`的功能，即能够显示出应用程序发出的系统调用，从而可以分析ucore应用的系统调用执行过程。
   - 答：在kernel中的```syscall```中根据```tf->tf_regs.reg_eax```（系统调用类型号）判断系统调用的类型，并使用```cprintf()```函数输出即可。
