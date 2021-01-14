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
