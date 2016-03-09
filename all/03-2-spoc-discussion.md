
（2）(spoc) 有一台假想的计算机，页大小（page size）为32 Bytes，支持32KB的虚拟地址空间（virtual address space）,有4KB的物理内存空间（physical memory），采用二级页表，一个页目录项（page directory entry ，PDE）大小为1 Byte,一个页表项（page-table entries
PTEs）大小为1 Byte，1个页目录表大小为32 Bytes，1个页表大小为32 Bytes。页目录基址寄存器（page directory base register，PDBR）保存了页目录表的物理地址（按页对齐）。

PTE格式（8 bit） :
```
  VALID | PFN6 ... PFN0
```
PDE格式（8 bit） :
```
  VALID | PT6 ... PT0
```
其
```
VALID==1表示，表示映射存在；VALID==0表示，表示映射不存在。
PFN6..0:页帧号
PT6..0:页表的物理基址>>5
```
在[物理内存模拟数据文件](./03-2-spoc-testdata.md)中，给出了4KB物理内存空间的值，请回答下列虚地址是否有合法对应的物理内存，请给出对应的pde index, pde contents, pte index, pte contents。
```

Virtual Address 6c74
Virtual Address 748b
```

答案：
```
Virtual Address 6c74:
  --> pde index : 0x1b pde contents:(valid 1,pfn 0x20)
    --> pte index : 0x03 pte contents :(valid 1 pfn 0x61)
      --> Translates to Physical Address 0xc34 --> Value: 06

Virtual Address 748b:
    --> pde index:0x1d pde contents:(valid 1, pfn 0x00)
        --> pte index:0x04 pte contents:(valid 0, pfn 0x7f)
            --> Fault (page table entry not valid)
```



（3）请基于你对原理课二级页表的理解，并参考Lab2建页表的过程，设计一个应用程序（可基于python, ruby, C, C++，LISP等）可模拟实现(2)题中描述的抽象OS，可正确完成二级页表转换。


#include <iostream>  
int memo[4096];//4KB内存 
uint32_t get_page(uint32_t v_addr) {
 	uint32_t ans = 0; 
	uint32_t pde = v_addr & 0x00007c00; 
	uint32_t pte = v_addr & 0x000003e0; 
	return ans; 
} 

using namespace std; 
int main(int argc, const char * argv[]) { 
	ifstream fin("1.txt"); 
	char s[100]; 
	int c = 0, num; 
	while (fin >> s) { 
		if (strcmp(s, "page") == 0) { 
			fin >> s; continue; 
		} 
		sscanf(s, "%x", &num); 
		memo[c] = num; c++; 
	} 
	int x; 
	while(true) { 
		scanf("%x", &x); 
		int offset = x %32; 
		x = x>>5; 
		int pte = x %32; 
		x = x>>5; 
		int pde = x %32; 
		int pdbr = 544; 
		int pde_2 = pdbr + pde; 
		int valid1 = memo[pde_2] >> 7; 
		if(valid1 == 0) { 
			printf("Fault (page directory entry not valid)\n"); 
			continue; 
		} 
		else { 
			int pter = memo[pde_2] % (1<<7); 
			printf("pde index : %x pde contents :(valid %d, pfn %x)\n",pde,valid1,pter); 
			int pte_2 = (pter << 5) + pte; 
			int pframe = memo[pte_2]; 
			int valid2 = pframe >> 7; 
			if(valid2 ==0) { 
				printf("Fault (page table entry not valid)\n");
				continue;
			} 
			else { 
				printf("pte index : %x pte contents :(valid %d, pfn %x)\n",pte,valid2,pframe %(1<<7)); 
				int pp = ((pframe %(1<<7)) << 5) + offset; 
				printf("%x\n",memo[pp]); 
			} 
		} 
		system("pause");
	} 
	return 0; 
}



--- 
