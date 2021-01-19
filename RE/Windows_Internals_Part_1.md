# Windows Internals Part 1 - 7th Ed.

## Chapter 1 - Concepts and Tools

### Windows Operating System Versions

#### Windows 10 and OneCore

- Windows 10 converged code bases for all Windows platforms (PC, IoT, XBox, etc.)
- Called **OneCore**

### Foundation Concepts and Terms

#### Windows API

- Win32 API is the 32-64bit user-mode programming interface (sometimes called Windows API)
  - Originally composed of c-style functions, got confusing, giving rise to *Component Object Model (COM)*
  - COM allows for programs to communicate & share data, called *Object Linking and Embedding (OLE)*
  - COM servers expose functions to programs, usually in *Dynamically Linked Libraries (DLLs)*
- WinRT exposes platform services for Windows Store Apps, is built on top of COM, can be used for many languages

#### The .NET Framework

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
  
#### Services, Functions, and Routines
  
  - Windows API Functions - Documented, callable subroutines in Win32 API
  - Native System Services (System Calls) - Undocumented, underlying OS services callable from user-mode
  - Kernel Support Functions/Routines - Subroutines in OS that can only be called from kernel mode
  - Windows Services - Processes started by Windows service control manager
  - Dynamic Link Libraries (DLLs) Callable subroutines packaged as binary file that can be dynamically loaded by applications
  
#### Processes
  
- Private Virtual Address Space - set of virtual addresses process can use
- Executable Program - initial code & data mapped into processes virtual address space
- List of Open Handles - Maps various system resources that are accessible to all threads
- Security Context - Access token that id's user, security groups, priviliges and more
- Process ID - Unique identifier, internally part of an identifier called client ID
- At Least One Thread of Execution - Technically optionally, but such a process would be largely useless
- Security context - stored in an access token object
- List of Open Handles - point to objects like files, shared memory sections or a synchronization object
- Virtual Address Descriptors (VADs) - data structures used by memory manager to track virtual addresses used by proccess

#### Threads

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

#### Fibers

- Allow applications to schedule it's own threads rather than having Windows do it
- Often called *lightweight threads*
- They are invisible to the kernel as they are implemented in user mode (in Kernel32.dll)
- Fibers can be composed of their own fibers

#### User-mode scheduling threads

- Only available on 64-bit Windows versions
- Similar to fibers, but visible to the kernel which allows them to issue blocking system calls and share resources
- Can switch contexts in user mode by yielding, thus not involving the scheduler
  - From kernel prespective, nothing has changed until an operation involving the kernel occurs
  - Once kernel involved, *directed context switch* occurs, and dedicated kernel-mode thread is switched to
- Concurrent UMS threads cannot run on multiple proccessors, but do still follow a pre-emptive model

#### Jobs

- Extension to the process model that already exists in Windows
- Allows gruops of proccesses to be managed and manipulated as a group
- Allows control of certain attributes and limits for associated processes, as well as records basic accounting info
- Compensates from lack of structured process tree in Windows, often more powerful that UNIX-style process trees

#### Virtual Memory

- Based on flat (linear) address space that provides the illusion of a large, private address space
  - Provides logical view of memory that may not correspond to its physical layout
- At runtime, memory manager translates virtual addresses to physical addresses
  - Allows OS to ensure processes don't bump into each other or the OS data
- Due to virtual memory being much more than physical, the manager *pages* some of the memory to disk
  - Paging frees physical memory for other proesses or the OS itself, bringing th paged data back when needed later
- Pages usually are fixed size (4k by default), though the pages are not necessarily contiguous
- Virtual address spaces vary by hardware
  - x86 (32 bit) - maximum VAD is 4GB, the lower half for user storage, the upper half for OS memory
    - The upper halves usually are the same, consisting of the OS's virtual memory
- Boot time options can alter this layout, for instance giving users 3GB and OS only 1 GB
  - Usually used for database applications to help prevent paging as much due to large volumes of data
- For large databases, *Address Windowing Extensions (AWE)* allow 32-bit applications to allocate 64 GB of physical memory
  - Views are mapped into the virtual address space, allowing parts of the physical memory to be accessed at a time
- x64 (64 bit) - 128 TB as of Window 8.1, though theoretically could go much higher

#### Kernel Mode vs. User Mode

- Windows uses two processor modes, even if architecture allows for more:
  - User Mode - application code
  - Kernel Mode - OS code (system services, drivers)
- On x86/64, there are 4 "rings" of privilege, but only 0 & 3 are used for kernel & user modes respectively
- User mode processes have their own virtual address space, kernel mode OS & device driver code share one virtual address space
- Pages are tagged to indicate the access mode the processor must be in to access them 
  - System space = kernel, user space = either
  - Pages can also be read-only (usually containing static content), and are not writable in either mode
- On processors with no-execute memory protection, Windows marks data pages as non-executable (Data Execution Protection (DEP))
- In kernel mode, there's no read/write protection for private system memory
  - Once in kernel mode, OS & device driver code can bypass Windows security access objects
- Driver code has complete access to the OS data, thus can be very high risk
  - Windows 2000 added a driver-signing mechanism to warn of unsigned plug-and-play drivers (though doesn't affect other drivers)
  - Driver Verifier also helps device dirver writers find bugs that can cause security issues
- On 64 bit and ARM versions of Windows 8.1, kernel-mode code-signing (KMCS) dictates all drivers must be signed
  - Users cannot explicitly force install of an unsigned driver, even as admin
- On Windows 10, all new drivers must be signed by only two accepted certification authorities, then by Microsoft
- Certain vendors/platroms/enterprise configs can customize there policies (such as Device Guard technology)
- User code must switch to kernel mode to make system calls, using a special processor instruction
  - A mode transition is *not* a context switch, thus doesn't affect thread scheduling *per se*
- Typically, more graphics intensive programs run in kernel mode
  - Advanced applications like Direct2D do usermode calculations then pass final results to kernel, so this may shift time spent

#### Hypervisor

- Specialized & highly privileged component that allows virtualization and isolation of all resources on the machine
- Has greater access than the kernel itself, allowing it to provide new services known as *virtualization-based security (VBS)*
  - Device Guard - Provides Hypervisor Code Integrity (HVCI) for code-signing guarentees & customization of signature policies
  - Hyper Guard - Protects key kernel-related and hypervisor-related data structures and code
  - Credential Guard - Prevents unauthorized access to domain account credentials & secrets
  - Application Guard - Provides strong sandbox for Microsoft Edge browser
  - Host Guardian and Shielded Fabric - Leverage a virtual TPM (v-TPM) to protect a vm from the infrastructure it's running on
- The key advantage is that these protections are not vulnerable to driver risks, signed or not
  - Virtual Trust Levels (VTLs) define leves of privilage in the hypervisor to enable this protection
    - VTL 0 - the OS and it's components (lower privilage)
    - VTL 1 - VBS technologies (higher privilage), thus cannot be affected by kernel code
  
#### Firmware

- Windows components typicall rely on security of lower level components, so where does the chain of trust start?
- Firmware that is UEFI-based provides a secure boot from the start
- Trusted Platform Module (TPM) can measure the process to provide attestation as well
- Windows can also ship firmware updates to help increase security at this critical juncture

#### Terminal Services and Multiple Sessions

- Terminal Services allow for multiple interactive user sessions on a single system
  - Allows for remote user sessions of desktops as well as single applications running on the server
- The first session is considered the services session (session zero, contains system service hosting processes)
  - First login session at the physical console is session one, then other remote sessions can be established
- Client Windows editions permit single user either remote or local, but not both at once
- Server Windows editions permit two simultaneous remote connections by default, or more if configured as terminal server
- Windows APIs exist to programmatically detect if program is running in terminal server session

#### Objects and Handles

- A *kernel object* is a single, run-time instance of a statically defined object type
- An *object type* comprises a system-defined data type, functions that operate on that data and a set of object attributes
  - Object types include process, file, thread, events, etc.
- An *object attribute* is a field of data in an object that partially defines the object's state
- An *object methods* manipulate objects, usually to read or change the object in question
- Unlike normal data structures, objects are completely opaque; you must use an object service to interact with them
- Objects, through the help of a kernel component called the *object manager*, provide means for:
  - Providing human-readable names for system resources
  - Sharing resources and data among processes
  - Protecting resources from unauthorized access
  - Reference tracking so that unused objects can be deallocated once no longer in use
- Not all data structures in Windows are objects, just those that need to be share, protected, named or made visible to users
- Structures only use by one component of the OS to implement internal functions are *not* objects

#### Security

- Core security capabilities of Windows include:
  - Discretionary and mandatory protection for all shareable objects
  - Security auditing for accountability of subjects, users and thier actions
  - User authentication at login
  - Prevention of one user from accessing unitialized resources
- Windows has three forms of access control over these objects:
  - Discretionary access control - Owners of objects can grant or deny access to them, usually by user attributes
  - Privileged access control - Allows admins can take control of files to if the owner is available
  - Mandatory integrity control - Additional level of security control for objects accessed from within same user account
- *AppContainer* is a sandbox for Windows Apps that isolate them in relation to other processes
- The Windows subsystem also implements object-based security, by placing security descriptors on shared objects

#### Registry

- System database that contains information required to boot & configure the system
- Provides a window into in-memory volatile data, such as current hardware state of the system & performance counters
- Editting the registry directly can be very hazardous, so exercise extreme caution

#### Unicode

- 16 bit wide characters used by Windows for format to store internal text strings
- Many functions have two entry points, one for Unicode (wide) and another for ASCII (narrow)
  - ASCII strings are transformed to Unicode before being used internally
  - ASCII support is mainly to run Windows 9x code
- COM-based APIs use Unicode strings typed as *BSTR*
  - Null-terminated array of Unicode chars with the length of the string in bytes stored in 4 bytes before the array in memory
  
### Digging into Windows Internals
