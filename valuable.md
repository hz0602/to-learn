### Some valuable information about xv6

1. The **wait** system call just makes the parent process blocked until it wait the moment that one of its child processes exits.
   If we want to make parent process blocked until all its child processes exit, we can use a loop of **wait**.

2. In the middle of pipe reading, we can utilize a while loop to make sure that we read all bytes from other side of the pipe,
   and **read** system call will return 0 if all the writing sides of the pipe close.
    
3. Each process has two stacks: a user stack and a kernel stack. When a process executes user instructions, it only uses the 
   user stack. But after a process enters the kernel, the kernel code executes in the process's kernel stack.

4. Something regarding syscall's process of execution:

   1. Every syscall interface is declared in **user/user.h**.
   2. Passing arguments such as syscall number and other parameters to kernel space through **entry** defined in **user/usys.pl**.
   3. Maybe **entry** will execute some assmblely or something, then it will jump to **syscall()** define in **kernel/syscall.c**.
   4. Now, it's in the kernel space.
   5. **syscall()** can reads syscall number(in **myproc()->trapframe->a7**) and other parameters from registers and it will
      call corresponding syscall(defined in **kernel/sysproc.c**) through syscall number.
   6. those concrete syscalls can read parameters through **argint, argaddr**.
   7. After returning from those concrete syscalls, **syscall()** can return a value through register **myproc()->trapframe->a0**.
   8. There is a way to exchange data between kernel space and user space through **copyout, copyin**

5. Something about file system:

   1. A file is conceptually an **inode**. But what we see in the terminal or file manager is some filenames, they are called **link**.
   2. An inode holds the **metadata** about a file, including its type, its length, the location of the file's content in disk, and the
     number of links to the inode.
   3. A **link** consists of an **entry** in a directory and it can point to an **inode**.
   4. Obviously, an inode can have several link.
   5. An **inode** and the disk space holding its content will be freed when the **inode's** link count is zero and no file descriptos
      refer to it.
   6. Hard link in linux: origin_link -> inode <- hard_link, only when all the hard links that point to the same inode are freed, the
      inode will be erased.
   7. Soft link in linux: soft_link -> origin_link -> inode, **referred through the filename**

6. Some thoughts about mmap (mapping between virtual address and physical address)

   1. the way of mapping of the kernel and a normal process is distinctly different.
   2. the mapping address of the kernel defined in **kernel/memlayout.h**
   3. Example: Lab "Speed up system calls"
     * There is a more virtual address called **USYSCALL** for kernel, it's uesd to map each scheduled process's physical space of the pid.
     * When each process is created, we assign it a more space for its pid, and we will set pagetable. So we map that address with
       the kernel's **USYSCALL**.
     * There is a register for the current schelduld process's pagetable, as we say above, through this pagetable we can use **USYSCALL**
       to read the scheduled process's pid.
     * Therefore, the speeding up syscall just read the current process'pid through kernel's **USYSCALL** address instead of being trapped into
          kernel space.

7. there kinds of event which cause transfer
   * system call
   * exception
   * interrupt
