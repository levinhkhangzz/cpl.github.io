---
title: "How to prepare for a low-level interview"
description: "What is the right mentality when preparing for a technical low-level interview and what to expect."
keywords: [low-level, technical, interview, coding, embedded, tutorial, training]
date: 2018-11-08
---


## Introduction
First thing you should do when pursing a career in low-level systems (*kernel development, embedded systems, drivers, firmware, boot loaders, etc.*) is to realise that all knowledge you have from high-level languages and systems will amount to almost nothing.

In other words you should prepare to change your views, mentality and approach when it comes to developing and designing low-level systems. When I was working on my own Operating System ([Classic](https://github.com/thee-engineer/classic)) I started to think and work in [base-16](https://en.wikipedia.org/wiki/Hexadecimal), I started to think about what seemed at the time the strangest optimisations, such as aligning my code to cache lines. Another thing that I found different by a mile, was debugging. When you work close to the metal and from scratch there aren’t many tools around to help you. No *asserts*, no *tests*, no *printf* function, until you implement them. So be ready to debug with what you have, and that might be a single LED if you are unlucky or if you are lucky [UART](https://en.wikipedia.org/wiki/Universal_asynchronous_receiver-transmitter) or [JTAG](https://en.wikipedia.org/wiki/JTAG) interfaces.

## Knowledge
I could go into a lot of detail about everything you should know but then I’d end up writing a book (there are books out there about Operating Systems and Embedded systems which are great starting points).

After deciding you have it in you, it’s time to load up on some technical concepts that you **must** definitely know before even touching a keyboard or [SoC](https://en.wikipedia.org/wiki/System_on_a_chip).

I’d personally say someone is ready to start diving into low-level development after they feel comfortable with the topics below and can answer these questions:

* What is an **Operating System**?
* What is a **kernel**?
* What is the difference between a kernel and an OS?
* What is “**memory**” and how many types of memory are there?
* How is a **CPU structured**?
* How is the CPU connected to the rest of the “computer”?
* What is the **Stack**? What is the **Heap**? When do you use each one?
* What is **Virtual Memory** and why do we have it?
* What are **interrupts** and where are they used?
* What are the CPU’s modes?
* What are **system calls** and where and why are they used?
* What is the difference between 32 and 64 bit architectures?
* What is **atomicity**?
* What is **concurrency**?
* What is a **pointer**?
* What are **signals**? What are they used for?
* What is the difference between pass by reference and pass by value?
* Are you comfortable with different hardware peripherals?
* What are **device mapped registers**? How do you access them?
* How does dynamic memory allocation work?
* What is a **memory leak**? Who is responsible for it?
* What is a **cache** and how does it work? How many cache types are there?
* Why is the userland split from the kernel space?
* What is a buffer/stack overflow?
* How would you design a scheduler?
* What is a **mutex**? or a **semaphore**? or a **spin-lock**?
* What is **starvation** and how does it occur?
* What is a **memory page**?

It is very likely that I missed some key concepts, but these questions should give you an idea of what you must know.

## Tools & Language
Now hopefully you have the right mindset, the knowledge and you need some tools to start working and showing off your ideas and designs.

There are two **must know** languages:
* C
* Assembly

### Assembly
There is no universal assembly language, each architecture has its own set of assembly instructions, but they share bits and pieces. I personally chose to learn **ARM Assembly** and a little bit of **Intel x86**. And even then, each architecture might have different versions of the instruction set, because as hardware changes so does the instruction set.

When picking an assembly language to learn, make sure to read about [RISC vs CISC](https://cs.stanford.edu/people/eroberts/courses/soco/projects/risc/risccisc/). It’s good to know and it should help you make an informed decision.

### C
I believe everybody is familiar with **C** and came across it at least once in their developer lifetime. Some might associate it with “weird” arcane things like `void*`, `SEGFAULTS`, `malloc` and `free`. Some might call it the language with no strings (`char*`). People are not wrong about **C**, it is different from high-level languages like **Python** or **JavaScript**, but **C** has its purpose and there is a reason why it passed the test of time. **C** is a robust language that doesn’t hold your hand, **C** is fast and it doesn’t hide things from you.

There is no **best programming language**, there is only **the right programming language** for the job. While you could develop an entire website in **C**, it wouldn’t be pleasant or a fun experience and you’d be better off doing it in **JavaScript**. This works the other way around too, You wouldn’t develop a real operating system in **JavaScript**.

#### Why C?

* As I said, it is fast, robust, doesn’t hold your hand and allows you freedom (to make amazing things or **disastrous mistakes**, up to you).
* It is another-level-up from the assembly level and you can see this by calling `gcc -s` when “compiling”. This tells `gcc` to only generate the assembly code for the current machine architecture.
* All big architectures have **C** compilers that allow your software to be compiled and executed on most machines.
* Trust the compiler, it can really optimise your code, especially with `-O1`, `-O2` or if you feel brave `-O3`. But you must help the compiler whenever you can and that is why knowing some assembly will help you do that.
* It allows pointer manipulation and logic, in other words you have direct access to the main memory.
* Builtin low-level concepts, such as `volatile`, `extern`, `__attribute__`, packed and/or aligned code/data-structures, etc.

## What to do now?

After you practice your **C** skills (you can do this on coding challenges websites: [HackerRank](https://www.hackerrank.com), [LeetCode](https://leetcode.com)) and feel happy with your skills, you can start practicing more low-level focused challenges.

Some in-between (low/high level) challenges I encountered on [LeetCode](https://leetcode.com) are:

* [String to Integer (atoi)](https://leetcode.com/problems/string-to-integer-atoi/description/)
* [Design Circular Queue](https://leetcode.com/problems/design-circular-queue/description/)
* [Power of Four](https://leetcode.com/problems/power-of-four/description/)
* [Number of 1 Bits](https://leetcode.com/problems/number-of-1-bits/description/)
* [Valid Parentheses](https://leetcode.com/problems/valid-parentheses/description/)
* [LRU Cache](https://leetcode.com/problems/lru-cache/description/)
* [Reverse Bits](https://leetcode.com/problems/reverse-bits/description/)
* [Reverse Polish Notation](https://leetcode.com/problems/evaluate-reverse-polish-notation/description/)

Similar to some of the above you can try and implement functions from the C standard library (just for practice, don’t do this in a serious program).


### Some more low-level focused questions that I came across are:

* Write a functions that returns the input with its endianness reversed

```rust
uint32_t swap_endian(uint32_t word);
```

* What is the output to the following section of code

```rust
int main() {
	int a = -64; float b = -7.0f;
	printf("%#X %#X", a, *(int*)&b);

	return 0;
}
```

* Write a function that can determine the endianness of a system at runtime
* More to come …

