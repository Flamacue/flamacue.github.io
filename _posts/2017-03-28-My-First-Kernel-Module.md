---
layout: post
title: "My First Kernel Module"
categories: journal
tags: [technology, linux, programming]
---

So I've been a linux user since my second year in college (2013-2014), and since then, I've absolutely fallen in love with *nix operating systems (don't worry, I'm not Joaquin Phoenix in the movie Her). The problem is that I've never really had the motivation to dive deeper into kernel space. In fact I've only really scraped the surface thanks to my systems programming class. But I love learning, so I said screw it, let's take a ride on the Linux Railroad!

Ok, if I'm being 100% honest, this wasn't just a random thought I had. I'm a subscriber of the subreddit r/linux, and a while ago someone posted [this talk](https://www.youtube.com/watch?v=0IQlpFWTFbM) about how you YES YOU TOO can be a kernel hacker. A corny title, I know, but the talk is actually pretty awesome! The speaker has such great energy, and gave me the great idea to `strace` commands when I have no internet on a long commute.

So let's get down to the dirty details of what I wrote. Are you ready for it? It truly is magical. Low and behold, my own Hello World kernel module!!!!

### Hello.c

```c

#include<stdio.h>
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Adam K. Sumner");
MODULE_DESCRIPTION("We all gotta start somewhere");

static int __init hello_init(void){
  printk(KERN_DEBUG "I've infiltrated the kernel!\n");
  return 0;
}

static void __exit hello_cleanup(void){
  printk(KERN_DEBUG "Goodbye, but mark my words. I'll be back!\n");
}

module_init(hello_init);
module_exit(hello_cleanup);

```

Ok so I may have pulled the bait and switch on you, since in reality, this module does nothing but print to the kernel's debug log level, but this is what I hope will be a gateway program into writing some really cool things, or really useless but funny things. I'll take either.

If you're still reading this, then you've made it to the fun part of the blog. Educational reading! Also here's a picture of a puppy, you've earned it.

![puppy](/images/puppy.PNG){:class="img-responsive"}

First let's talk about what a kernel module even is. I'll assume you already know what an operating system kernel is/does. A kernel module is a piece of code that can be used to expand upon the linux kernel without having to recompile your kernel. They can be loaded in and out of memory at the sudoer or root user's will. Most commonly, loadable kernel modules take the form of drivers.

That's it. Pretty simple, huh? For our purposes, it's just a way to write kernel code without having to actually dig through the millions of lines of code of the actual kernel. I'm not the bravest of souls, but one day, I'll get my feet wet.

So let's talk about some relevant Unix commands.

`lsmod` -- lists all of the currently loaded kernel modules in your operating system. Open up a terminal and check out the output!

```bash
$~ lsmod
Module			Size  Used by
ehci_hcd               73728  1 ehci_pci
xhci_hcd              172032  1 xhci_pci
usb_common             16384  1 usbcore
i8042                  28672  0
serio                  20480  4 serio_raw,atkbd,i8042
sdhci_acpi             16384  0
sdhci                  40960  1 sdhci_acpi
led_class              16384  3 sdhci,input_leds,ath9k
mmc_core              122880  2 sdhci,sdhci_acpi
```
The first column is the module name, the second column is the size of the module, and the third column tells you what other modules are using it. You'll have more output in your terminal than this, trust me.

`modinfo` -- lists information of a specific module. In my kernel module, do you remember seeing these lines?
```c

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Adam K. Sumner");
MODULE_DESCRIPTION("We all gotta start somewhere");

```

Well running `modinfo` on my compiled kernel module shows us this output

```

filename:       /home/adam/kernel_modules/hello_world/hello.ko
description:    We all gotta start somewhere
author:         Adam K. Sumner
license:        GPL
depends:        
vermagic:       4.9.16-1-MANJARO SMP preempt mod_unload modversions 

```

See anything familiar? ;)

`insmod` -- loads the kernel module. 

Of course anyone who has written C code knows that you need to compile it before you can actually do anything with it(this ain't Python folks). After installing the linux headers on our machine, we can successfully create a Makefile to compile our program and output our `hello.ko` file.

### Makefile
```bash

KDIR?=/lib/modules/$(shell uname -r)/build/

obj-m += hello.o

all:
	make -C $(KDIR)  M=$(PWD) modules

clean:
	make -C $(KDIR) M=$(PWD) clean

```

If you run `insmod hello.ko` in the terminal and do a `lsmod | grep hello`, I think you'll see a familiar guy.

```

$~ sudo insmod hello.ko
$~ modinfo | grep hello
hello                  16384  0

```

But how do we see what the init function did? It didn't print to the console because this ain't `printf` folks. `printk` is special function for the kernel that lets you specify a log level. It prints to the kernel log message buffer. If we type `dmesg | tail -1` we'll have the ability to read what was just printed.

```

$~ demsg | tail -1
[11065.361938] I've infiltrated the kernel!

```

`rmmod` -- unloads a kernel module. Not much to be said here. Instead of the init function being invoked, now our cleanup function gets invoked. Following the same steps as we did with `insmod`, we can check to see what our module did.

```

$~ sudo rmmod hello
$~ dmesg | tail -1
[11679.775611] Goodbye, but mark my words. I'll be back!

```

Cool stuff right?! Ok not really THIS stuff, but knowing these basics is crucial to being able to develop useful kernel modules.

If you're interested in writing your own kernel modules check out [The Eudyptula Challenge](http://www.eudyptula-challenge.org). But hurry and sign up fast because once they reach 20,000 people, the challenge will be closed to the public. It's a cool website that sends you tasks to complete via email involving kernel development. It also trains you to submit your solutions the true Linux way. Once I'm finished with the challenge, hopefully I can write my very official patch. 

Overall I'm happy that I've decided to embark on this journey to kernel space (cooler than kerbal space). I think it will end up improving my every day coding skills, and who knows, it may spark some new systems programming interests I didn't know I had in me.
