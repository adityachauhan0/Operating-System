![[Pasted image 20240816153608.png]]
#### **Overview**
- **Prerequisite Knowledge**: Understanding of how a computer program runs is essential for this course.
- **Von Neumann Model**: 
  - A program executes by fetching, decoding, and executing instructions sequentially.
  - Modern processors enhance this process with techniques like out-of-order execution and parallelism.

#### **The Role of the Operating System (OS)**
- **Primary Function**: The OS makes the system easier to use by managing resources like the CPU, memory, and disks.
- **Virtualization**: 
  - The OS transforms physical resources into more versatile virtual resources.
  - This process allows multiple programs to run concurrently, sharing system resources.
  
- **Interfaces**: 
  - The OS provides interfaces (APIs) for applications to interact with these virtualized resources.
  - Hundreds of system calls are typically available for tasks such as running programs, allocating memory, and accessing files.

- **Resource Management**: 
  - The OS manages system resources efficiently and fairly.
  - It is also known as a resource manager due to its role in overseeing the CPU, memory, and disks.

#### **Example Code**
- **Purpose**: A simple C program is presented to illustrate a basic operation.
- **Function**: The code continuously loops, printing a string provided as a command-line argument.

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/time.h>
#include <assert.h>
#include "common.h"

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "usage: cpu <string>\n");
        exit(1);
    }
    char *str = argv[1];
    while (1) {
        Spin(1);
        printf("%s\n", str);
    }
    return 0;
}
```

#### **Key Concepts**
- **Virtualization**: Making physical resources easier to use by abstracting them into virtual resources.
- **APIs**: Tools provided by the OS for applications to perform tasks.
- **Resource Manager**: The OS's role in efficiently allocating system resources among multiple programs.
### Summary: Virtualizing the CPU
#### **Introduction**
- **Simple Program Example**:
  - The program `cpu.c` repeatedly calls the `Spin()` function, waits for a second, and prints a user-provided string.
  - The program runs indefinitely until manually terminated by the user (e.g., using "Control-c").

- **Running the Program**:
  - When run on a system with a single CPU, the program simply prints the string at one-second intervals.
  
  ```bash
  prompt> gcc -o cpu cpu.c -Wall
  prompt> ./cpu "A"
  A A A A
  ˆC
  ```

- **Running Multiple Instances**:
  - When running multiple instances of the same program, they appear to run simultaneously, even on a single processor.
  
  ```bash
  prompt> ./cpu A & ; ./cpu B & ; ./cpu C & ; ./cpu D &
  A B D C A B D C A C B D ...
  ```
  
#### **Concept of CPU Virtualization**
- **Illusion of Parallelism**:
  - Even with one CPU, the OS creates the illusion that multiple programs are running concurrently by virtualizing the CPU.
  - This is achieved by the OS, often with hardware support, creating virtual CPUs that allow multiple programs to run as if they each had their own processor.

- **APIs and User Interaction**:
  - The OS provides APIs for users to run, manage, and control multiple programs. These APIs are a key way users interact with the OS.

- **Resource Management**:
  - Running multiple programs simultaneously introduces the challenge of deciding which program gets to run at any given time.
  - The OS uses policies to manage these decisions, balancing system resources among competing programs.

#### **Example Code: Memory Access**
- **Memory Management Example**:
  - The program in Figure 2.3 demonstrates simple memory allocation, using `malloc` to allocate memory, and then continuously updating and printing the value stored at that memory location.
  
  ```c
  #include <unistd.h>
  #include <stdio.h>
  #include <stdlib.h>
  #include "common.h"
  
  int main(int argc, char *argv[]) {
      int *p = malloc(sizeof(int)); // a1
      assert(p != NULL);
      printf("(%d) address of p: %08x\n", getpid(), (unsigned) p); // a2
      *p = 0; // a3
      while (1) {
          Spin(1);
          *p = *p + 1;
          printf("(%d) p: %d\n", getpid(), *p); // a4
      }
      return 0;
  }
  ```

  - **Key Concepts**:
    - `malloc` is used for dynamic memory allocation.
    - `getpid()` retrieves the process ID, and the address and value of the allocated memory are printed in a loop.

#### **Key Takeaways**
- **CPU Virtualization**:
  - The OS can virtualize a single physical CPU to appear as multiple virtual CPUs, allowing multiple programs to run concurrently.
- **Resource Management**:
  - Managing which program runs at a given time is a critical function of the OS, handled through specific policies.
- **User Interaction**:
  - APIs provided by the OS enable users to control and interact with the running programs.

This summary provides a concise overview of the main ideas and concepts related to CPU virtualization and resource management introduced in the text.

### Summary: Virtualizing Memory

#### **Introduction to Memory Access**
- **Physical Memory Model**:
  - Memory in modern machines is represented as an array of bytes.
  - Programs access memory by specifying an address to read or write data.
  - Memory is constantly accessed during program execution, for both data structures and instruction fetches.

- **Program Example**:
  - A simple program (`mem.c`) is provided to demonstrate memory allocation and manipulation.
  
  ```c
  #include <unistd.h>
  #include <stdio.h>
  #include <stdlib.h>
  #include "common.h"
  
  int main(int argc, char *argv[]) {
      int *p = malloc(sizeof(int)); // a1
      assert(p != NULL);
      printf("(%d) address of p: %08x\n", getpid(), (unsigned) p); // a2
      *p = 0; // a3
      while (1) {
          Spin(1);
          *p = *p + 1;
          printf("(%d) p: %d\n", getpid(), *p); // a4
      }
      return 0;
  }
  ```

#### **Output and Analysis**
- **Single Instance**:
  - When running a single instance of the program, it allocates memory using `malloc()`, prints the memory address, and updates the value at that address in a loop.
  - Example output:
  
  ```bash
  prompt> ./mem
  (2134) memory address of p: 00200000
  (2134) p: 1
  (2134) p: 2
  (2134) p: 3
  ˆC
  ```

- **Multiple Instances**:
  - When running multiple instances of the same program, each instance allocates memory at the same virtual address (`00200000`), but updates the value independently.
  
  ```bash
  prompt> ./mem &; ./mem &
  (24113) memory address of p: 00200000
  (24114) memory address of p: 00200000
  (24113) p: 1
  (24114) p: 1
  (24114) p: 2
  (24113) p: 2
  ...
  ```

#### **Concept of Memory Virtualization**
- **Virtual Address Space**:
  - The OS provides each process with its own private virtual address space, isolating it from other processes.
  - This makes it appear as though each process has access to the same memory address (e.g., `00200000`), but in reality, each process is working with its own virtual memory, mapped to the physical memory by the OS.

- **Memory Management**:
  - The OS is responsible for mapping virtual addresses to physical memory, ensuring that each process has its own isolated environment.
  - This prevents one process from interfering with the memory of another, creating the illusion that each process has exclusive access to memory.

#### **Key Takeaways**
- **Memory Access**:
  - Programs access memory using addresses, but through virtualization, these addresses are mapped by the OS to physical memory.
- **Virtualization**:
  - The OS virtualizes memory, giving each process a private virtual address space, even when physical memory is shared.
- **Process Isolation**:
  - Memory virtualization is a key technique that ensures processes remain isolated from one another, enhancing both security and stability.

This summary provides an overview of the concepts related to memory virtualization and how the OS manages memory for multiple processes.
### Summary: Concurrency

#### **Introduction to Concurrency**
- **Concept of Concurrency**:
  - Concurrency refers to the challenges that arise when multiple tasks are performed simultaneously within the same program.
  - Originally, concurrency issues were a concern primarily for the operating system, as it had to manage multiple processes at once.
  - In modern computing, concurrency problems also occur in multi-threaded programs, where multiple threads execute within the same memory space.

#### **Concurrency in Operating Systems**
- **Example Program**:
  - A simple multi-threaded program (`thread.c`) is provided to illustrate concurrency issues.
  
  ```c
  #include <stdio.h>
  #include <stdlib.h>
  #include "common.h"
  
  volatile int counter = 0;
  int loops;
  
  void *worker(void *arg) {
      int i;
      for (i = 0; i < loops; i++) {
          counter++;
      }
      return NULL;
  }
  
  int main(int argc, char *argv[]) {
      if (argc != 2) {
          fprintf(stderr, "usage: threads <value>\n");
          exit(1);
      }
      loops = atoi(argv[1]);
      pthread_t p1, p2;
      printf("Initial value : %d\n", counter);
  
      Pthread_create(&p1, NULL, worker, NULL);
      Pthread_create(&p2, NULL, worker, NULL);
      Pthread_join(p1, NULL);
      Pthread_join(p2, NULL);
      printf("Final value : %d\n", counter);
      return 0;
  }
  ```

- **Expected vs. Actual Output**:
  - The program creates two threads that each increment a shared counter (`counter`) a specified number of times (`loops`).
  - When the program is run with `loops` set to 1000, the expected final value of `counter` is 2000.

  ```bash
  prompt> ./thread 1000
  Initial value : 0
  Final value : 2000
  ```

- **Unexpected Behavior**:
  - When running the program with a larger value for `loops` (e.g., 100,000), the final value of `counter` is incorrect and varies between runs.
  
  ```bash
  prompt> ./thread 100000
  Initial value : 0
  Final value : 143012 // Unexpected value
  prompt> ./thread 100000
  Initial value : 0
  Final value : 137298 // Different incorrect value
  ```

#### **Concurrency Challenges**
- **Explanation of the Issue**:
  - The unexpected results occur because the increment operation on `counter` is not atomic.
  - The operation involves three steps: loading the value from memory, incrementing it, and storing it back. If these steps are interleaved between threads, inconsistent results occur.

- **Atomicity and Concurrency**:
  - The lack of atomicity in critical sections of code leads to race conditions, where the outcome depends on the interleaving of operations between threads.
  - This issue is central to concurrency and requires careful handling in multi-threaded programs to ensure correct behavior.

#### **Key Takeaways**
- **Concurrency Issues**:
  - Concurrency introduces complex problems when multiple threads or processes operate on shared resources simultaneously.
  - The order of operations in a non-atomic sequence can lead to unpredictable results, as demonstrated by the variable outcomes in the example program.
  
- **Importance of Proper Concurrency Handling**:
  - To build correct concurrent programs, developers must understand the primitives provided by the OS and hardware, such as locks, semaphores, and atomic operations.
  - Proper synchronization mechanisms are essential to prevent race conditions and ensure that concurrent programs function correctly.

This summary provides an overview of the key concepts related to concurrency, including the challenges it presents and the importance of atomicity in multi-threaded programming.
### Summary: Persistence

#### **Introduction to Persistence**
- **Concept of Persistence**:
  - Persistence refers to the ability to store data in a way that it survives beyond the life of a process, particularly when power is lost or the system crashes.
  - Unlike system memory (e.g., DRAM), which is volatile and loses data when power is off, persistent storage ensures that data is retained even after such events.
  - This is crucial for users who depend on the operating system (OS) to manage and store their data reliably.

#### **Persistence in Operating Systems**
- **Hardware and Software**:
  - **Hardware**: Persistent storage is typically provided by I/O devices like hard drives or solid-state drives (SSDs), which store long-lived data.
  - **Software**: The file system, a component of the OS, manages how data is stored on these devices. It ensures that files are stored in a reliable and efficient manner.

- **File System**:
  - Unlike CPU and memory virtualization, the OS does not virtualize storage; instead, it provides shared access to files across different processes.
  - Example: When writing a C program, you create a source file using an editor, compile it, and then run the executable. The file system manages this flow of data across different processes (e.g., editor, compiler, and runtime).

#### **Persistence Example Program**
- **Basic File Operations**:
  - A simple program in Figure 2.6 demonstrates basic file operations: opening a file, writing to it, and closing it.

  ```c
  #include <stdio.h>
  #include <unistd.h>
  #include <assert.h>
  #include <fcntl.h>
  #include <sys/types.h>
  
  int main(int argc, char *argv[]) {
      int fd = open("/tmp/file", O_WRONLY | O_CREAT | O_TRUNC,
       S_IRWXU);
      assert(fd > -1);
      int rc = write(fd, "hello world\n", 13);
      assert(rc == 13);
      close(fd);
      return 0;
  }
  ```

  - **`open()`**: Opens or creates a file.
  - **`write()`**: Writes data to the file.
  - **`close()`**: Closes the file, signaling that no more data will be written.

- **OS Role in Persistence**:
  - The OS handles these file operations through system calls routed to the file system.
  - The file system determines where the data will reside on the disk and updates its internal structures to track this information.
  
#### **Challenges and Solutions in Persistence**
- **File System Complexity**:
  - Writing to disk involves more than just placing data at a specific location; the file system must manage data placement, maintain metadata, and ensure data integrity.
  - Device drivers, which are specialized OS code, interact with the hardware to perform these operations.

- **Performance Considerations**:
  - File systems often delay writes to optimize performance by batching operations.
  - Techniques like **journaling** or **copy-on-write** are employed to ensure data integrity in case of system failures during write operations. These methods carefully order write operations to allow recovery to a consistent state after a crash.

- **Data Structures and Access Methods**:
  - File systems use a variety of data structures, from simple lists to complex B-trees, to manage files and directories efficiently.
  - The choice of data structure and access method depends on the specific needs of the file system, such as supporting fast lookups, updates, or ensuring robustness against failures.

#### **Key Takeaways**
- **Importance of Persistence**:
  - Persistence is critical for ensuring that data is not lost in the event of power failures or system crashes.
  - The file system is the OS component responsible for managing persistent data, requiring sophisticated techniques to handle performance and reliability challenges.

- **Complexity of Managing Persistent Storage**:
  - The OS must perform complex operations to manage storage devices and ensure that data is stored and retrieved efficiently and reliably.
  - Understanding these operations and the associated challenges is essential for developing and maintaining reliable file systems. 

This summary provides an overview of the key concepts related to persistence, emphasizing the importance of reliable data storage and the role of the OS and file systems in managing persistent data.
### 2.5 Design Goals

In designing an operating system (OS), it's crucial to establish clear goals that guide the system's development and help in making necessary trade-offs. These goals include creating abstractions to make the system convenient and user-friendly. Abstractions are essential in computer science because they allow complex tasks to be broken down into manageable parts. For example, writing in a high-level language like C without worrying about assembly code is possible due to abstraction.

Another goal is to provide high performance, meaning the OS should minimize overheads, such as additional time or space. While achieving perfect performance is challenging, striving for solutions that reduce overheads is important. Additionally, protection is a key goal, ensuring that applications are isolated from each other and from the OS itself, preventing malicious or accidental actions from causing harm. This protection is essential for maintaining system integrity, especially when multiple programs run simultaneously.

Reliability is another critical goal, as the OS must run continuously without failure. With the increasing complexity of modern OSes, building a reliable system is challenging but vital. Energy efficiency, security, and support for mobility are also important goals, especially as OSes are used on a wide range of devices, from servers to smartphones. These goals may vary depending on the system's intended use, but the principles of building an OS are generally applicable across different platforms.

### 2.6 Some History

The development of operating systems (OS) has evolved significantly over time. Initially, OSes were merely libraries of commonly used functions, and batch processing was the norm, where a human operator would manage and run jobs one at a time. As OSes advanced, they took on a more central role in managing machines, particularly with the introduction of protection mechanisms. For instance, system calls, pioneered by the Atlas computing system, allowed the OS to manage resources securely by transitioning control from user mode to kernel mode, where the OS has full access to the system hardware.

The era of multiprogramming marked a significant advancement, with OSes supporting multiple jobs simultaneously to improve CPU utilization. This required innovations in memory protection, concurrency handling, and system reliability. UNIX, introduced by Ken Thompson and Dennis Ritchie at Bell Labs, became a pivotal OS, known for its simplicity, power, and support for small, powerful programs that could be combined to accomplish larger tasks.

In the modern era, personal computers (PCs) became prevalent, but early PC OSes like DOS lacked many of the advanced features of minicomputer OSes, such as memory protection and proper job scheduling. However, over time, these features were integrated into modern OSes like Mac OS X and Windows NT, leading to more robust and secure systems. The rise of Linux, driven by Linus Torvalds, further democratized OS development and adoption, especially in server environments and mobile platforms like Android.

UNIX's influence remains strong, with its principles and ideas continuing to shape modern OSes. Despite challenges and legal battles over its ownership, UNIX's legacy endures, making it a foundational element in today's computing landscape.

### 2.7 Summary

Operating systems have evolved to make computer systems more user-friendly and efficient. Modern OSes have been influenced by historical developments, incorporating key concepts like virtualization, concurrency, and persistence. While this book covers many important topics, such as CPU and memory virtualization, device and file system management, and concurrency, it doesn't delve deeply into networking, graphics, or advanced security topics. Nonetheless, by the end of this course, you'll gain a deeper understanding of how computer systems work and appreciate the complexities involved in OS design and implementation. Now, it's time to dive into the details and start learning!
