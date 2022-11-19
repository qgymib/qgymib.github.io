---
title: longjmp的行为
date: 2021-08-02T08:53:22+08:00
tags:
---

## 起因

在造轮子 [call_on_stack](https://github.com/qgymib/call_on_stack) 时，发现 `longjmp()` 函数所作的工作不像表面这么简单。

<!-- more -->

[call_on_stack](https://github.com/qgymib/call_on_stack) 支持在用户态随意进行堆栈切换，由于涉及到栈操作，因此测试涉及栈操作的posix函数兼容性是必要的。其中一个使用 `longjmp()` 的用例如下：

```c
#include <setjmp.h>

jmp_buf g_env;
char buffer_1[10240];
char buffer_2[10240];

static void _func_2(void* arg)
{
    /* 跳转 */
    longjmp(g_env, 1);
}

static void _func_1(void* arg)
{
    /* 在内存区块 #buffer_2 中调用函数 #_func_2() */
    call_on_stack(buffer_2, sizeof(buffer_2), _func_2, NULL);
}

int main(int argc, char* argv[])
{
    /* 保存上下文 */
    if (setjmp(g_env) != 0)
    {
        return 0;
    }

    /* 在内存区块 #buffer_1 中调用函数 #_func_1() */
    call_on_stack(buffer_1, sizeof(buffer_1), _func_1, NULL);

    return 0;
}

```

如上用例很简单，逻辑上讲如果 `call_on_stack` 函数实现没有问题，那么此用例应该能够通过才对。然而实际测试发现，此测试用例会导致core dump。


## 分析

基本原理我已经在 [call_on_stack#limit](https://github.com/qgymib/call_on_stack#limit) 中有过阐述，这里我们从源码层面分析下。

### glibc行为分析

影响glibc行为的主要是 [shadow stack](https://en.wikipedia.org/wiki/Shadow_stack) 机制。我们从glibc-2.32入手分析一下 `setjmp()` 与 `longjmp()` 的行为。

#### 源码分析

在 `x86-64` 平台下，我们可以看到 `setjmp()` 函数保存了当前线程的 `shadow Stack Pointer`:

```assembly
/* setjmp for x86-64.
   Copyright (C) 2001-2020 Free Software Foundation, Inc.
   This file is part of the GNU C Library.

   The GNU C Library is free software; you can redistribute it and/or
   modify it under the terms of the GNU Lesser General Public
   License as published by the Free Software Foundation; either
   version 2.1 of the License, or (at your option) any later version.

   The GNU C Library is distributed in the hope that it will be useful,
   but WITHOUT ANY WARRANTY; without even the implied warranty of
   MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
   Lesser General Public License for more details.

   You should have received a copy of the GNU Lesser General Public
   License along with the GNU C Library; if not, see
   <https://www.gnu.org/licenses/>.  */

#include <sysdep.h>
#include <jmpbuf-offsets.h>
#include <jmp_buf-ssp.h>
#include <asm-syntax.h>
#include <stap-probe.h>

/* Don't save shadow stack register if shadow stack isn't enabled.  */
#if !SHSTK_ENABLED
# undef SHADOW_STACK_POINTER_OFFSET
#endif

ENTRY (__sigsetjmp)
	/* Save registers.  */
	movq %rbx, (JB_RBX*8)(%rdi)
#ifdef PTR_MANGLE
# ifdef __ILP32__
	/* Save the high bits of %rbp first, since PTR_MANGLE will
	   only handle the low bits but we cannot presume %rbp is
	   being used as a pointer and truncate it.  Here we write all
	   of %rbp, but the low bits will be overwritten below.  */
	movq %rbp, (JB_RBP*8)(%rdi)
# endif
	mov %RBP_LP, %RAX_LP
	PTR_MANGLE (%RAX_LP)
	mov %RAX_LP, (JB_RBP*8)(%rdi)
#else
	movq %rbp, (JB_RBP*8)(%rdi)
#endif
	movq %r12, (JB_R12*8)(%rdi)
	movq %r13, (JB_R13*8)(%rdi)
	movq %r14, (JB_R14*8)(%rdi)
	movq %r15, (JB_R15*8)(%rdi)
	lea 8(%rsp), %RDX_LP	/* Save SP as it will be after we return.  */
#ifdef PTR_MANGLE
	PTR_MANGLE (%RDX_LP)
#endif
	movq %rdx, (JB_RSP*8)(%rdi)
	mov (%rsp), %RAX_LP	/* Save PC we are returning to now.  */
	LIBC_PROBE (setjmp, 3, LP_SIZE@%RDI_LP, -4@%esi, LP_SIZE@%RAX_LP)
#ifdef PTR_MANGLE
	PTR_MANGLE (%RAX_LP)
#endif
	movq %rax, (JB_PC*8)(%rdi)

#ifdef SHADOW_STACK_POINTER_OFFSET
# if IS_IN (libc) && defined SHARED && defined FEATURE_1_OFFSET
	/* Check if Shadow Stack is enabled.  */
	testl $X86_FEATURE_1_SHSTK, %fs:FEATURE_1_OFFSET
	jz L(skip_ssp)
# else
	xorl %eax, %eax
# endif
	/* Get the current Shadow-Stack-Pointer and save it.  */
	rdsspq %rax
	movq %rax, SHADOW_STACK_POINTER_OFFSET(%rdi)
# if IS_IN (libc) && defined SHARED && defined FEATURE_1_OFFSET
L(skip_ssp):
# endif
#endif
#if IS_IN (rtld)
	/* In ld.so we never save the signal mask.  */
	xorl %eax, %eax
	retq
#else
	/* Make a tail call to __sigjmp_save; it takes the same args.  */
	jmp __sigjmp_save
#endif
END (__sigsetjmp)
hidden_def (__sigsetjmp)

```

而 `longjmp()` 读取了当前线程的 `Shadow Stack Pointer` 并进行校验：

```assembly
/* Copyright (C) 2001-2020 Free Software Foundation, Inc.
   This file is part of the GNU C Library.

   The GNU C Library is free software; you can redistribute it and/or
   modify it under the terms of the GNU Lesser General Public
   License as published by the Free Software Foundation; either
   version 2.1 of the License, or (at your option) any later version.

   The GNU C Library is distributed in the hope that it will be useful,
   but WITHOUT ANY WARRANTY; without even the implied warranty of
   MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
   Lesser General Public License for more details.

   You should have received a copy of the GNU Lesser General Public
   License along with the GNU C Library; if not, see
   <https://www.gnu.org/licenses/>.  */

#include <sysdep.h>
#include <jmpbuf-offsets.h>
#include <jmp_buf-ssp.h>
#include <asm-syntax.h>
#include <stap-probe.h>

/* Don't restore shadow stack register if
   1. Shadow stack isn't enabled.  Or
   2. __longjmp is defined for __longjmp_cancel.
 */
#if !SHSTK_ENABLED || defined __longjmp
# undef SHADOW_STACK_POINTER_OFFSET
#endif

/* Jump to the position specified by ENV, causing the
   setjmp call there to return VAL, or 1 if VAL is 0.
   void __longjmp (__jmp_buf env, int val).  */
	.text
ENTRY(__longjmp)
	/* Restore registers.  */
	mov (JB_RSP*8)(%rdi),%R8_LP
	mov (JB_RBP*8)(%rdi),%R9_LP
	mov (JB_PC*8)(%rdi),%RDX_LP
#ifdef PTR_DEMANGLE
	PTR_DEMANGLE (%R8_LP)
	PTR_DEMANGLE (%R9_LP)
	PTR_DEMANGLE (%RDX_LP)
# ifdef __ILP32__
	/* We ignored the high bits of the %rbp value because only the low
	   bits are mangled.  But we cannot presume that %rbp is being used
	   as a pointer and truncate it, so recover the high bits.  */
	movl (JB_RBP*8 + 4)(%rdi), %eax
	shlq $32, %rax
	orq %rax, %r9
# endif
#endif
#ifdef SHADOW_STACK_POINTER_OFFSET
# if IS_IN (libc) && defined SHARED && defined FEATURE_1_OFFSET
	/* Check if Shadow Stack is enabled.  */
	testl $X86_FEATURE_1_SHSTK, %fs:FEATURE_1_OFFSET
	jz L(skip_ssp)
# else
	xorl %eax, %eax
# endif
	/* Check and adjust the Shadow-Stack-Pointer.  */
	/* Get the current ssp.  */
	rdsspq %rax
	/* And compare it with the saved ssp value.  */
	subq SHADOW_STACK_POINTER_OFFSET(%rdi), %rax
	je L(skip_ssp)
	/* Count the number of frames to adjust and adjust it
	   with incssp instruction.  The instruction can adjust
	   the ssp by [0..255] value only thus use a loop if
	   the number of frames is bigger than 255.  */
	negq %rax
	shrq $3, %rax
	/* NB: We saved Shadow-Stack-Pointer of setjmp.  Since we are
	       restoring Shadow-Stack-Pointer of setjmp's caller, we
	       need to unwind shadow stack by one more frame.  */
	addq $1, %rax

	movl $255, %ebx
L(loop):
	cmpq %rbx, %rax
	cmovb %rax, %rbx
	incsspq %rbx
	subq %rbx, %rax
	ja L(loop)

L(skip_ssp):
#endif
	LIBC_PROBE (longjmp, 3, LP_SIZE@%RDI_LP, -4@%esi, LP_SIZE@%RDX_LP)
	/* We add unwind information for the target here.  */
	cfi_def_cfa(%rdi, 0)
	cfi_register(%rsp,%r8)
	cfi_register(%rbp,%r9)
	cfi_register(%rip,%rdx)
	cfi_offset(%rbx,JB_RBX*8)
	cfi_offset(%r12,JB_R12*8)
	cfi_offset(%r13,JB_R13*8)
	cfi_offset(%r14,JB_R14*8)
	cfi_offset(%r15,JB_R15*8)
	movq (JB_RBX*8)(%rdi),%rbx
	movq (JB_R12*8)(%rdi),%r12
	movq (JB_R13*8)(%rdi),%r13
	movq (JB_R14*8)(%rdi),%r14
	movq (JB_R15*8)(%rdi),%r15
	/* Set return value for setjmp.  */
	mov %esi, %eax
	mov %R8_LP,%RSP_LP
	movq %r9,%rbp
	LIBC_PROBE (longjmp_target, 3,
		    LP_SIZE@%RDI_LP, -4@%eax, LP_SIZE@%RDX_LP)
	jmpq *%rdx
END (__longjmp)

```

如此，由于 `call_on_stack` 并未针对 `shadow stack` 进行适配，因此若 `setjmp()` / `longjmp()` 开启了 `shadow stack` 功能，就会导致测试用例崩溃。

#### 解决方法

我们在 `glibc` 的 `makecontext()` 方法中能够找到应对方法。参考如下片段：

```c
#if SHSTK_ENABLED
  struct pthread *self = THREAD_SELF;
  unsigned int feature_1 = THREAD_GETMEM (self, header.feature_1);
  /* NB: We must check feature_1 before accessing __ssp since caller
	 may be compiled against ucontext_t without __ssp.  */
  if ((feature_1 & X86_FEATURE_1_SHSTK) != 0)
    {
      /* Shadow stack is enabled.  We need to allocate a new shadow
         stack.  */
      unsigned long ssp_size = (((uintptr_t) sp
				 - (uintptr_t) ucp->uc_stack.ss_sp)
				>> STACK_SIZE_TO_SHADOW_STACK_SIZE_SHIFT);
      /* Align shadow stack to 8 bytes.  */
      ssp_size = ALIGN_UP (ssp_size, 8);

      ucp->__ssp[1] = ssp_size;
      ucp->__ssp[2] = ssp_size;

      /* Call __push___start_context to allocate a new shadow stack,
	 push __start_context onto the new stack as well as the new
	 shadow stack.  NB: After __push___start_context returns,
	   ucp->__ssp[0]: The new shadow stack pointer.
	   ucp->__ssp[1]: The base address of the new shadow stack.
	   ucp->__ssp[2]: The size of the new shadow stack.
       */
      __push___start_context (ucp);
    }
  else
#endif
```

我们可以看到，当开启 `shadow stack` 使能时，`makecontext()` 函数做了两件事情：
1. 检查线程本身是否开启了ssp功能；
2. 若线程开启了ssp功能，则构造一个新的 shadow stack。

### msvc行为分析

[Unicorn Devblog: setjmp/longjmp on Windows](https://blog.lazym.io/2020/09/21/Unicorn-Devblog-setjmp-longjmp-on-Windows/) 已经说的很清楚了，不再赘述。
