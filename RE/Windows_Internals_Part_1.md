# Windows Internals Part 1 - 7th Ed.

## Chapter 1 - Concepts and Tools

### Windows 10 and OneCore

- Windows 10 converged code bases for all Windows platforms (PC, IoT, XBox, etc.)
- Called **OneCore**

### Windows API

- Win32 API is the 32-64bit user-mode programming interface (sometimes called Windows API)
  - Originally composed of c-style functions, got confusing, giving rise to *Component Object Model (COM)*
  - COM allows for programs to communicate & share data, called *Object Linking and Embedding (OLE)*
  - COM servers expose functions to programs, usually in *Dynamically Linked Libraries (DLLs)*
- WinRT exposes platform services for Windows Store Apps, is built on top of COM, can be used for many languages

### The .NET Framework

- The .Net framework is composed of two parts:
  - Common Language Runtime (CLR) - runtime engine, implemented as COM server (DLL)
  - .Net Framework Class Library (FCL) - collection of types for client/server applications
  
  OS Mode | .NET Component
  :------:|:-------------:
  User Mode (Managed Code) | .NET App (EXE)
  User Mode (Managed Code) | FCL (.NET Assemblies) (DLLs)
  User Mode (Native Code) | CLR (COM DLL Server)
  User Mode (Native Code) | Windows API DLLs
  Kernal Mode | Windows Kernel
  
  ### Services, Functions, and Routines
  
  - Windows API Functions - Documented, callable subroutines in Win32 API
  - Native System Services (System Calls) - Undocumented, underlying OS services callable from user-mode
  - Kernel Support Functions/Routines - Subroutines in OS that can only be called from kernel mode
  - Windows Services - Processes started by Windows service control manager
  - Dynamic Link Libraries (DLLs) Callable subroutines packaged as binary file that can be dynamically loaded by applications
  
  ### Processes
  
  - Private Virtual Address Space - set of virtual addresses process can use
  - Executable Program - initial code & data mapped into processes virtual address space
  - List of Open Handles - Maps various system resources that are accessible to all threads
  - Security Context - Access token that id's user, security groups, priviliges and more
  - Process ID - Unique identifier, internally part of an identifier called client ID
  - At Least One Thread of Execution - Technically optionally, but such a process would be largely useless
  - Security context - stored in an access token object
  - List of Open Handles - point to objects like files, shared memory sections or a synchronization object
  - Virtual Address Descriptors (VADs) - data structures used by memory manager to track virtual addresses used by proccess

### Threads

- Entity within proccess that Windows schedules for execution, made up of:
  - Contents of a set of CPU registers representing the state of the processor
  - Two stacks; one for kernel mode, the other for user mode
  - Private storage called *thread-local storage (TLS)*
  - A unique identifier called a *thread ID* (thread and proccess IDs never overlap)
- Threads can also have their own security context to impersonate that of their clients
- A thread's context is composed of it's volatile & non-volatile registers and private storage
  - This structure is architecture specific
- Two mechanisms are used to reduce cost of switching threads: *fibers and user-mode scheduling (UMS)*
- Threads share the processes virtual address space & other resources
  - This means all threads have full permissions & accesss to whole proccess virtual address space
- Cannot access other virtual address spaces however (unless they are mapped into it's private address space)
  - A process may also be able to access another proccess to use cross-process memory functions

### Fibers

- Allow applications to schedule it's own threads rather than having Windows do it
- Often called *lightweight threads*
- They are invisible to the kernel as they are implemented in user mode (in Kernel32.dll)
- Fibers can be composed of their own fibers

### User-mode scheduling threads

- Only available on 64-bit Windows versions
- Similar to fibers, but visible to the kernel which allows them to issue blocking system calls and share resources
- Can switch contexts in user mode by yielding, thus not involving the scheduler
  - From kernel prespective, nothing has changed until an operation involving the kernel occurs
  - Once kernel involved, *directed context switch* occurs, and dedicated kernel-mode thread is switched to
- Concurrent UMS threads cannot run on multiple proccessors, but do still follow a pre-emptive model

### Jobs

- Extension to the process model that already exists in Windows
- Allows gruops of proccesses to be managed and manipulated as a group
- Allows control of certain attributes and limits for associated processes, as well as records basic accounting info
- Compensates from lack of structured process tree in Windows, often more powerful that UNIX-style process trees

### Virtual Memory

