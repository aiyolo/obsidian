---
uid: 
aliases: null
created: 2023-01-12 11:26:11
updated: 2023-03-08 15:30:58
tags:
  - gdb
  - clip
  - cpp
source: https://stackoverflow.com/questions/5497855/what-does-value-optimized-out-mean-in-gdb
title: Gdb 中 optimized out 是什么意思
dg-publish: true
---

# Gdb 中 optimized out 是什么意思

> [!summary]  
> - 带优化时，编译器使用寄存器存储参数变量，如使用 edi 存储第一个参数，如果一个函数内部中间调用了其他函数，那么 edi 会用于存储其他函数的参数，而本函数的参数就会覆盖掉，此时打印本函数的参数变量就会产生 optimized out。  
> - 不带优化时，参数变量的值会存到栈中，因此打印变量的值仍然可以读出结果来

## 原文链接

**Minimal runnable example with disassembly analysis**

As usual, I like to see some disassembly to get a better understanding of what is going on.

In this case, the insight we obtain is that if a variable is optimized to be stored [only in a register rather than the stack](https://stackoverflow.com/questions/4584089/what-is-the-function-of-the-push-pop-instructions-used-on-registers-in-x86-ass/33583134#33583134), and then the register it was in gets overwritten, then it shows as `<optimized out>` as [mentioned by R.](https://stackoverflow.com/questions/5497855/what-does-value-optimized-out-mean-in-gdb#comment6239178_5497906).

Of course, this can only happen if the variable in question is not needed anymore, otherwise the program would lose its value. Therefore it tends to happen that at the start of the function you can see the variable value, but then at the end it becomes `<optimized out>`.

One typical case which we often are interested in of this is that of the function arguments themselves, since these are:

- always defined at the start of the function
- may not get used towards the end of the function as more intermediate values are calculated.
- tend to get overwritten by further function subcalls which must setup the exact same registers to satisfy the calling convention

This understanding actually has a concrete application: when using [reverse debugging](https://stackoverflow.com/questions/1206872/how-to-go-to-the-previous-line-in-gdb/46996380#46996380), you might be able to recover the value of variables of interest simply by stepping back to their last point of usage: [How do I view the value of an < optimized out> variable in C++?](https://stackoverflow.com/questions/9123676/how-do-i-view-the-value-of-an-optimized-out-variable-in-c/60562034#60562034)

main.c

```
#include <stdio.h>

int __attribute__((noinline)) f3(int i) {
    return i + 1;
}

int __attribute__((noinline)) f2(int i) {
    return f3(i) + 1;
}

int __attribute__((noinline)) f1(int i) {
    int j = 1, k = 2, l = 3;
    i += 1;
    j += f2(i);
    k += f2(j);
    l += f2(k);
    return l;
}

int main(int argc, char *argv[]) {
    printf("%d\n", f1(argc));
    return 0;
}
```

Compile and run:

```
gcc -ggdb3 -O3 -std=c99 -Wall -Wextra -pedantic -o main.out main.c
gdb -q -nh main.out
```

Then inside GDB, we have the following session:

```
Breakpoint 1, f1 (i=1) at main.c:13
13          i += 1;
(gdb) disas
Dump of assembler code for function f1:
=> 0x00005555555546c0 <+0>:     add    $0x1,%edi
   0x00005555555546c3 <+3>:     callq  0x5555555546b0 <f2>
   0x00005555555546c8 <+8>:     lea    0x1(%rax),%edi
   0x00005555555546cb <+11>:    callq  0x5555555546b0 <f2>
   0x00005555555546d0 <+16>:    lea    0x2(%rax),%edi
   0x00005555555546d3 <+19>:    callq  0x5555555546b0 <f2>
   0x00005555555546d8 <+24>:    add    $0x3,%eax
   0x00005555555546db <+27>:    retq   
End of assembler dump.
(gdb) p i
$1 = 1
(gdb) p j
$2 = 1
(gdb) n
14          j += f2(i);
(gdb) disas
Dump of assembler code for function f1:
   0x00005555555546c0 <+0>:     add    $0x1,%edi
=> 0x00005555555546c3 <+3>:     callq  0x5555555546b0 <f2>
   0x00005555555546c8 <+8>:     lea    0x1(%rax),%edi
   0x00005555555546cb <+11>:    callq  0x5555555546b0 <f2>
   0x00005555555546d0 <+16>:    lea    0x2(%rax),%edi
   0x00005555555546d3 <+19>:    callq  0x5555555546b0 <f2>
   0x00005555555546d8 <+24>:    add    $0x3,%eax
   0x00005555555546db <+27>:    retq   
End of assembler dump.
(gdb) p i
$3 = 2
(gdb) p j
$4 = 1
(gdb) n
15          k += f2(j);
(gdb) disas
Dump of assembler code for function f1:
   0x00005555555546c0 <+0>:     add    $0x1,%edi
   0x00005555555546c3 <+3>:     callq  0x5555555546b0 <f2>
   0x00005555555546c8 <+8>:     lea    0x1(%rax),%edi
=> 0x00005555555546cb <+11>:    callq  0x5555555546b0 <f2>
   0x00005555555546d0 <+16>:    lea    0x2(%rax),%edi
   0x00005555555546d3 <+19>:    callq  0x5555555546b0 <f2>
   0x00005555555546d8 <+24>:    add    $0x3,%eax
   0x00005555555546db <+27>:    retq   
End of assembler dump.
(gdb) p i
$5 = <optimized out>
(gdb) p j
$6 = 5
(gdb) n
16          l += f2(k);
(gdb) disas
Dump of assembler code for function f1:
   0x00005555555546c0 <+0>:     add    $0x1,%edi
   0x00005555555546c3 <+3>:     callq  0x5555555546b0 <f2>
   0x00005555555546c8 <+8>:     lea    0x1(%rax),%edi
   0x00005555555546cb <+11>:    callq  0x5555555546b0 <f2>
   0x00005555555546d0 <+16>:    lea    0x2(%rax),%edi
=> 0x00005555555546d3 <+19>:    callq  0x5555555546b0 <f2>
   0x00005555555546d8 <+24>:    add    $0x3,%eax
   0x00005555555546db <+27>:    retq   
End of assembler dump.
(gdb) p i
$7 = <optimized out>
(gdb) p j
$8 = <optimized out>
```

To understand what is going on, remember from the x86 Linux calling convention: [What are the calling conventions for UNIX & Linux system calls on i386 and x86-64](https://stackoverflow.com/questions/2535989/what-are-the-calling-conventions-for-unix-linux-system-calls-on-i386-and-x86-6) you should know that:

- RDI contains the first argument
- RDI can get destroyed in function calls
- RAX contains the return value

From this we deduce that:

```
add    $0x1,%edi
```

corresponds to the:

```
i += 1;
```

**since `i` is the first argument of `f1`, and therefore stored in RDI.**

> i 存在 rdi

Now, while we were at both:

```
i += 1;
j += f2(i);
```

the value of RDI hadn't been modified, and therefore GDB could just query it at anytime in those lines.

However, as soon as the `f2` call is made:

- the value of `i` is not needed anymore in the program
- `lea 0x1(%rax),%edi` does `EDI = j + RAX + 1`, which both:
	- initializes `j = 1`
	- sets up the first argument of the next `f2` call to `RDI = j`

Therefore, when the following line is reached:

```
k += f2(j);
```

both of the following instructions have/may have modified RDI, which is the only place `i` was being stored (`f2` may use it as a scratch register, and `lea` definitely set it to RAX + 1):

```
   0x00005555555546c3 <+3>:     callq  0x5555555546b0 <f2>
   0x00005555555546c8 <+8>:     lea    0x1(%rax),%edi
```

and so RDI does not contain the value of `i` anymore. In fact, the value of `i` was completely lost! Therefore the only possible outcome is:

```
$3 = <optimized out>
```

A similar thing happens to the value of `j`, although `j` only becomes unnecessary one line later afer the call to `k += f2(j);`.

Thinking about `j` also gives us some insight on how smart GDB is. Notably, at `i += 1;`, the value of `j` had not yet materialized in any register or memory address, and GDB must have known it based solely on debug information metadata.

**`-O0` analysis**

If we use `-O0` instead of `-O3` for compilation:

```
gcc -ggdb3 -O0 -std=c99 -Wall -Wextra -pedantic -o main.out main.c
```

then the disassembly would look like:

```
11      int __attribute__((noinline)) f1(int i) {
=> 0x0000555555554673 <+0>:     55      push   %rbp
   0x0000555555554674 <+1>:     48 89 e5        mov    %rsp,%rbp
   0x0000555555554677 <+4>:     48 83 ec 18     sub    $0x18,%rsp
   0x000055555555467b <+8>:     89 7d ec        mov    %edi,-0x14(%rbp)

12          int j = 1, k = 2, l = 3;
   0x000055555555467e <+11>:    c7 45 f4 01 00 00 00    movl   $0x1,-0xc(%rbp)
   0x0000555555554685 <+18>:    c7 45 f8 02 00 00 00    movl   $0x2,-0x8(%rbp)
   0x000055555555468c <+25>:    c7 45 fc 03 00 00 00    movl   $0x3,-0x4(%rbp)

13          i += 1;
   0x0000555555554693 <+32>:    83 45 ec 01     addl   $0x1,-0x14(%rbp)

14          j += f2(i);
   0x0000555555554697 <+36>:    8b 45 ec        mov    -0x14(%rbp),%eax
   0x000055555555469a <+39>:    89 c7   mov    %eax,%edi
   0x000055555555469c <+41>:    e8 b8 ff ff ff  callq  0x555555554659 <f2>
   0x00005555555546a1 <+46>:    01 45 f4        add    %eax,-0xc(%rbp)

15          k += f2(j);
   0x00005555555546a4 <+49>:    8b 45 f4        mov    -0xc(%rbp),%eax
   0x00005555555546a7 <+52>:    89 c7   mov    %eax,%edi
   0x00005555555546a9 <+54>:    e8 ab ff ff ff  callq  0x555555554659 <f2>
   0x00005555555546ae <+59>:    01 45 f8        add    %eax,-0x8(%rbp)

16          l += f2(k);
   0x00005555555546b1 <+62>:    8b 45 f8        mov    -0x8(%rbp),%eax
   0x00005555555546b4 <+65>:    89 c7   mov    %eax,%edi
   0x00005555555546b6 <+67>:    e8 9e ff ff ff  callq  0x555555554659 <f2>
   0x00005555555546bb <+72>:    01 45 fc        add    %eax,-0x4(%rbp)

17          return l;
   0x00005555555546be <+75>:    8b 45 fc        mov    -0x4(%rbp),%eax

18      }
   0x00005555555546c1 <+78>:    c9      leaveq 
   0x00005555555546c2 <+79>:    c3      retq 
```

From this horrendous disassembly, we see that the value of RDI is moved to the stack at the very start of program execution at:

> rdi 移到了栈

```
mov    %edi,-0x14(%rbp)
```

and it then gets retrieved from memory into registers whenever needed, e.g. at:

```
14          j += f2(i);
   0x0000555555554697 <+36>:    8b 45 ec        mov    -0x14(%rbp),%eax
   0x000055555555469a <+39>:    89 c7   mov    %eax,%edi
   0x000055555555469c <+41>:    e8 b8 ff ff ff  callq  0x555555554659 <f2>
   0x00005555555546a1 <+46>:    01 45 f4        add    %eax,-0xc(%rbp)
```

The same basically happens to `j` which gets immediately pushed to the stack when when it is initialized:

```
   0x000055555555467e <+11>:    c7 45 f4 01 00 00 00    movl   $0x1,-0xc(%rbp)
```

Therefore, it is easy for GDB to find the values of those variables at any time: they are always present in memory!

This also gives us some insight on why it is not possible to avoid `<optimized out>` in optimized code: since the number of registers is limited, the only way to do that would be to actually push unneeded registers to memory, which would partly defeat the benefit of `-O3`.

**Extend the lifetime of `i`**

If we edited `f1` to return `l + i` as in:

```
int __attribute__((noinline)) f1(int i) {
    int j = 1, k = 2, l = 3;
    i += 1;
    j += f2(i);
    k += f2(j);
    l += f2(k);
    return l + i;
}
```

then we observe that this effectively extends the visibility of `i` until the end of the function.

This is because with this we force GCC to use an extra variable to keep `i` around until the end:

```
   0x00005555555546c0 <+0>:     lea    0x1(%rdi),%edx
   0x00005555555546c3 <+3>:     mov    %edx,%edi
   0x00005555555546c5 <+5>:     callq  0x5555555546b0 <f2>
   0x00005555555546ca <+10>:    lea    0x1(%rax),%edi
   0x00005555555546cd <+13>:    callq  0x5555555546b0 <f2>
   0x00005555555546d2 <+18>:    lea    0x2(%rax),%edi
   0x00005555555546d5 <+21>:    callq  0x5555555546b0 <f2>
   0x00005555555546da <+26>:    lea    0x3(%rdx,%rax,1),%eax
   0x00005555555546de <+30>:    retq
```

which the compiler does by storing `i += i` in RDX at the very first instruction.

Tested in Ubuntu 18.04, GCC 7.4.0, GDB 8.1.0.

---

> When you begin to touch your heart or let your heart be touched, you begin to discover that it's bottomless.  
> — <cite>Pema Chödrön</cite>
