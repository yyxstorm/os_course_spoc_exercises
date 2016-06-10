请分析em.c，并补充cpu.md中描述不够或错误的地方。包括：在v9-cpu中如何实现时钟中断的；v9-cpu指令，关键变量描述有误或不全的情况；在v9-cpu中的跳转相关操作是如何实现的；在v9-cpu中如何设计相应指令，可有效实现函数调用与返回；emhello/os0/os1等程序被加载到内存的哪个位置,其堆栈是如何设置的；在v9-cpu中如何完成一次内存地址的读写的；在v9-cpu中如何实现分页机制；
时钟中断：
•	每一个时钟周期，若设置了timeout，则timer+=delta，若timer>=timeout 会把trap设为mtimer 并且goto到interrupt执行有关中断的指令。
关键变量：
•	ssp:系统堆栈指针
•	usp:用户堆栈指针
•	cycle:循环计数
•	xcycle:4*cycle
•	timer:计时器
•	timeout：计时器到时间后，跳转到interrupt
•	delta：时钟周期长度
跳转相关操作：
•	JMP:跳转到operand0
•	JMPI:跳转懂啊operand0加上一个常量
•	JSR：跳转到operand0并保存PC
•	JSRA：跳转到operand0加上一个常量，并保存PC
•	还有很多Branch相关操作可以进行条件跳转
函数调用和返回：
•	利用JSR、JSRA跳转到相应函数位置实现调用，此时原先的PC被保存到寄存器，执行完函数后再跳转到寄存器所保存的值即可返回。函数调用跳转前参数被保存在堆栈内。
程序加载到内存哪个位置：
•	程序加载到内存起始位置，注意到688行read函数中的(void*)mem。
•	堆栈从底部向上进行。
内存地址的读写：
•	先获得虚拟地址，右移12位获得虚拟页号，在tr/tw数组中寻找对应的物理页号，若没有则调用rlook/vlook函数寻找对应的物理页号。得到物理页号后，将虚拟地址翻译成物理地址，然后根据offset获得对应的位置进行读写操作。
分页机制：
•	每页的大小是4096bit，
•	TLB中有4个转换表分别是内核和用户的读写转换表。
•	tr，tw指针指向用户或内核的读写表。
	
请编写一个小程序，在v9-cpu下，能够接收你输入的字符并输出你输入的字符
#include <u.h>

ivec(void *isr)
{
  asm(LL,8);
  asm(IVEC);
}

inout()
{
  asm(BIN);
  asm(LBA);
  asm(LI,1);
  asm(BOUT);
}

trap()
{
  inout(0);
  asm(RTI);
}

main()
{
  ivec(trap);
  asm(STI);
  while(1);
  asm(HALT);
}
请编写一个小程序，在v9-cpu下，能够产生各种异常/中断
//question2 test interruption

#include <u.h>

enum { // page table entry flags
  PTE_P   = 0x001,       // Present
  PTE_W   = 0x002,       // Writeable
  PTE_U   = 0x004,       // User
//PTE_PWT = 0x008,       // Write-Through
//PTE_PCD = 0x010,       // Cache-Disable
  PTE_A   = 0x020,       // Accessed
  PTE_D   = 0x040,       // Dirty
//PTE_PS  = 0x080,       // Page Size
//PTE_MBZ = 0x180,       // Bits must be zero
};

enum { // processor fault codes
  FMEM,   // bad physical address
  FTIMER, // timer interrupt
  FKEYBD, // keyboard interrupt
  FPRIV,  // privileged instruction
  FINST,  // illegal instruction
  FSYS,   // software trap
  FARITH, // arithmetic trap
  FIPAGE, // page fault on opcode fetch
  FWPAGE, // page fault on write
  FRPAGE, // page fault on read
  USER=16 // user mode exception 
};

char pg_mem[6 * 4096]; // page dir + 4 entries + alignment

int *pg_dir, *pg0, *pg1, *pg2, *pg3;

int current;

int in(port)    { asm(LL,8); asm(BIN); }
out(port, val)  { asm(LL,8); asm(LBL,16); asm(BOUT); }
ivec(void *isr) { asm(LL,8); asm(IVEC); }
lvadr()         { asm(LVAD); }
stmr(int val)   { asm(LL,8); asm(TIME); }
pdir(value)     { asm(LL,8); asm(PDIR); }
spage(value)    { asm(LL,8); asm(SPAG); }
halt(value)     { asm(LL,8); asm(HALT); }

void *memcpy() { asm(LL,8); asm(LBL, 16); asm(LCL,24); asm(MCPY); asm(LL,8); }
void *memset() { asm(LL,8); asm(LBLB,16); asm(LCL,24); asm(MSET); asm(LL,8); }
void *memchr() { asm(LL,8); asm(LBLB,16); asm(LCL,24); asm(MCHR); }

write(fd, char *p, n) { while (n--) out(fd, *p++); }

int strlen(char *s) { return memchr(s, 0, -1) - (void *)s; }

enum { BUFSIZ = 32 };
int vsprintf(char *s, char *f, va_list v)
{
  char *e = s, *p, c, fill, b[BUFSIZ];
  int i, left, fmax, fmin, sign;

  while (c = *f++) {
    if (c != '%') { *e++ = c; continue; }
    if (*f == '%') { *e++ = *f++; continue; }
    if (left = (*f == '-')) f++;
    fill = (*f == '0') ? *f++ : ' ';
    fmin = sign = 0; fmax = BUFSIZ;
    if (*f == '*') { fmin = va_arg(v,int); f++; } else while ('0' <= *f && *f <= '9') fmin = fmin * 10 + *f++ - '0';
    if (*f == '.') { if (*++f == '*') { fmax = va_arg(v,int); f++; } else for (fmax = 0; '0' <= *f && *f <= '9'; fmax = fmax * 10 + *f++ - '0'); }
    if (*f == 'l') f++;
    switch (c = *f++) {
    case 0: *e++ = '%'; *e = 0; return e - s;
    case 'c': fill = ' '; i = (*(p = b) = va_arg(v,int)) ? 1 : 0; break;
    case 's': fill = ' '; if (!(p = va_arg(v,char *))) p = "(null)"; if ((i = strlen(p)) > fmax) i = fmax; break;
    case 'u': i = va_arg(v,int); goto c1;
    case 'd': if ((i = va_arg(v,int)) < 0) { sign = 1; i = -i; } c1: p = b + BUFSIZ-1; do { *--p = ((uint)i % 10) + '0'; } while (i = (uint)i / 10); i = (b + BUFSIZ-1) - p; break;
    case 'o': i = va_arg(v,int); p = b + BUFSIZ-1; do { *--p = (i & 7) + '0'; } while (i = (uint)i >> 3); i = (b + BUFSIZ-1) - p; break;
    case 'p': fill = '0'; fmin = 8; c = 'x';
    case 'x': case 'X': c -= 33; i = va_arg(v,int); p = b + BUFSIZ-1; do { *--p = (i & 15) + ((i & 15) > 9 ? c : '0'); } while (i = (uint)i >> 4); i = (b + BUFSIZ-1) - p; break;
    default: *e++ = c; continue;
    }
    fmin -= i + sign;
    if (sign && fill == '0') *e++ = '-';
    if (!left && fmin > 0) { memset(e, fill, fmin); e += fmin; }
    if (sign && fill == ' ') *e++ = '-';
    memcpy(e, p, i); e += i;
    if (left && fmin > 0) { memset(e, fill, fmin); e += fmin; }
  }
  *e = 0;
  return e - s;
}

int printf(char *f) { static char buf[4096]; return write(1, buf, vsprintf(buf, f, &f)); } // XXX remove static from buf

trap(int c, int b, int a, int fc, int pc)
{
  printf("TRAP: ");
  switch (fc) {
  case FINST:  printf("FINST"); break;
  case FRPAGE: printf("FRPAGE [0x%08x]",lvadr()); break;
  case FWPAGE: printf("FWPAGE [0x%08x]",lvadr()); break;
  case FIPAGE: printf("FIPAGE [0x%08x]",lvadr()); break;
  case FSYS:   printf("FSYS"); break;
  case FARITH: printf("FARITH"); break;
  case FMEM:   printf("FMEM [0x%08x]",lvadr()); break;
  case FTIMER: printf("FTIMER"); current = 1; stmr(0); break;
  case FKEYBD: printf("FKEYBD [%c]", in(0)); break;
  default:     printf("other [%d]",fc); break;
  }
}

alltraps()
{
  asm(PSHA);
  asm(PSHB);
  asm(PSHC);
  trap();
  asm(POPC);
  asm(POPB);
  asm(POPA);
  asm(RTI);
}

setup_paging()
{
  int i;
  
  pg_dir = (int *)((((int)&pg_mem) + 4095) & -4096);
  pg0 = pg_dir + 1024;
  pg1 = pg0 + 1024;
  pg2 = pg1 + 1024;
  pg3 = pg2 + 1024;
  
  pg_dir[0] = (int)pg0 | PTE_P | PTE_W | PTE_U;  // identity map 16M
  pg_dir[1] = (int)pg1 | PTE_P | PTE_W | PTE_U;
  pg_dir[2] = (int)pg2 | PTE_P | PTE_W | PTE_U;
  pg_dir[3] = (int)pg3 | PTE_P | PTE_W | PTE_U;
  for (i=4;i<1024;i++) pg_dir[i] = 0;
  
  for (i=0;i<4096;i++) pg0[i] = (i<<12) | PTE_P | PTE_W | PTE_U;  // trick to write all 4 contiguous pages
  
  pdir(pg_dir);
  spage(1);
}

main()
{
  int t, d; 
  
  current = 0;
  ivec(alltraps);
  
  asm(STI);
  
  printf("test timer...");
  t = 0;
  stmr(10000);
  while (!current) t++;
  printf("(t=%d)...ok\n",t);
  
  printf("test invalid instruction...");
//  asm(_dd); // XXX find a better way
  asm(-1);
  printf("...ok\n");
  
  printf("test bad physical address...");
  t = *(int *)0x20000000;
  printf("...ok\n");

  printf("test divide by zero...");
  t = 10; d = 0; t /= d;
  printf("...ok\n");
  
  printf("test paging...");
  // reposition stack within first 16M
  asm(LI, 4*1024*1024); // a = 4M
  asm(SSP); // sp = a
  setup_paging();
  printf("identity map...ok\n");

  printf("test page fault read...");
  pg0[50] = 0;
  pdir(pg_dir);
  t = *(int *)(50<<12);
  printf("...ok\n");

  printf("test page fault write...");
  *(int *)(50<<12) = 5;
  printf("...ok\n");

  halt(0);
}
请编写一个小程序，在v9-cpu下，能够统计并显示内存大小

#include <u.h>

enum { // processor fault codes
  FMEM,   // bad physical address
  FTIMER, // timer interrupt
  FKEYBD, // keyboard interrupt
  FPRIV,  // privileged instruction
  FINST,  // illegal instruction
  FSYS,   // software trap
  FARITH, // arithmetic trap
  FIPAGE, // page fault on opcode fetch
  FWPAGE, // page fault on write
  FRPAGE, // page fault on read
  USER=16 // user mode exception
};

int in(port)    { asm(LL,8); asm(BIN); }
out(port, val)  { asm(LL,8); asm(LBL,16); asm(BOUT); }
ivec(void *isr) { asm(LL,8); asm(IVEC); }
halt(value)     { asm(LL,8); asm(HALT); }

void *memcpy() { asm(LL,8); asm(LBL, 16); asm(LCL,24); asm(MCPY); asm(LL,8); }
void *memset() { asm(LL,8); asm(LBLB,16); asm(LCL,24); asm(MSET); asm(LL,8); }
void *memchr() { asm(LL,8); asm(LBLB,16); asm(LCL,24); asm(MCHR); }

write(fd, char *p, n) { while (n--) out(fd, *p++); }

int strlen(char *s) { return memchr(s, 0, -1) - (void *)s; }

enum { BUFSIZ = 32 };

int vsprintf(char *s, char *f, va_list v)
{
  char *e = s, *p, c, fill, b[BUFSIZ];
  int i, left, fmax, fmin, sign;

  while (c = *f++) {
    if (c != '%') { *e++ = c; continue; }
    if (*f == '%') { *e++ = *f++; continue; }
    if (left = (*f == '-')) f++;
    fill = (*f == '0') ? *f++ : ' ';
    fmin = sign = 0; fmax = BUFSIZ;
    if (*f == '*') { fmin = va_arg(v,int); f++; } else while ('0' <= *f && *f <= '9') fmin = fmin * 10 + *f++ - '0';
    if (*f == '.') { if (*++f == '*') { fmax = va_arg(v,int); f++; } else for (fmax = 0; '0' <= *f && *f <= '9'; fmax = fmax * 10 + *f++ - '0'); }
    if (*f == 'l') f++;
    switch (c = *f++) {
    case 0: *e++ = '%'; *e = 0; return e - s;
    case 'c': fill = ' '; i = (*(p = b) = va_arg(v,int)) ? 1 : 0; break;
    case 's': fill = ' '; if (!(p = va_arg(v,char *))) p = "(null)"; if ((i = strlen(p)) > fmax) i = fmax; break;
    case 'u': i = va_arg(v,int); goto c1;
    case 'd': if ((i = va_arg(v,int)) < 0) { sign = 1; i = -i; } c1: p = b + BUFSIZ-1; do { *--p = ((uint)i % 10) + '0'; } while (i = (uint)i / 10); i = (b + BUFSIZ-1) - p; break;
    case 'o': i = va_arg(v,int); p = b + BUFSIZ-1; do { *--p = (i & 7) + '0'; } while (i = (uint)i >> 3); i = (b + BUFSIZ-1) - p; break;
    case 'p': fill = '0'; fmin = 8; c = 'x';
    case 'x': case 'X': c -= 33; i = va_arg(v,int); p = b + BUFSIZ-1; do { *--p = (i & 15) + ((i & 15) > 9 ? c : '0'); } while (i = (uint)i >> 4); i = (b + BUFSIZ-1) - p; break;
    default: *e++ = c; continue;
    }
    fmin -= i + sign;
    if (sign && fill == '0') *e++ = '-';
    if (!left && fmin > 0) { memset(e, fill, fmin); e += fmin; }
    if (sign && fill == ' ') *e++ = '-';
    memcpy(e, p, i); e += i;
    if (left && fmin > 0) { memset(e, fill, fmin); e += fmin; }
  }
  *e = 0;
  return e - s;
}

int printf(char *f) { static char buf[4096]; return write(1, buf, vsprintf(buf, f, &f)); }
int getmemsize(){asm(MSIZ);}//get memory size

main()
{
  printf("MEMSize: %d\n", getmemsize());
  asm(LI,0);
  asm(HALT);
}
