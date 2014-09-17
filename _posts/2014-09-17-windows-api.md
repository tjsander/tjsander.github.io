---
layout: post
title: "Processes, Threads and Synchronization in the Windows API"
image: https://farm1.staticflickr.com/35/115834464_07e911074c_b.jpg
permalink: windows-api.html
tags: code windows
---

## Introduction

Although in 2014 Windows-based mobile devices only account for 2.5% of all mobile traffic, approximately one in three web servers runs a Windows variant. Currently, Windows-based devices represent roughly 59% of all web traffic, and (by very rough estimate) 80% of all personal computer operating systems.[^fn-1]


One major implementation difference is the Windows API’s liberal use of parameters. Where the POSIX implementation might have two or three parameters, the Windows equivalent has six to ten. This stems from a design decision to make most of these functions boolean. Most API calls return nonzero on success and zero on failure. The lack of a return value means that all return values must be passed by reference as parameters. Depending on preference and style, this can be considered a benefit or a burden.


## Windows Processes

The primary difference between Windows processes and POSIX processes seems to be that Windows processes have an explicit primary thread of execution.[^fn-3] This suggests that on the kernel-level Windows treats all process execution as threads. The overhead in creating new processes in Windows appears to be greater than in a POSIX environment, thus Windows relies heavily on threaded application programming rather than multi-process programming for parallelization.[^fn-4]

### Creating Processes



{% highlight C %}
{% endhighlight %}

In the above example, you can see the initialization of startupinfo and process information objects. These containers are passed as parameters to createprocess along with default (null) values and the command line argv[1].

The eqivalent call for getpid is GetCurrentProcessID, while the parent ID can be found with a call to Process32. Windows API naming conventions seem to value specificity over simplicity.

### Ending Processes

The Windows call for a process to exit is ExitProcess, and the call to kill another process is TerminateProcess. As in POSIX, the terminated processes should be waited on by the parent process, though it is not mandatory. The equivalent calls to wait and waitpid are handled by WaitForSingleObject, which I will discuss in the synchronization section at greater length. WaitForSingleObject and WaitForMultipleObjects are the primary methods for all process and thread synchronization in Windows.



## Threads

Thread operation in Windows works similarly to the POSIX implementation. The equivalent call to pthread create is CreateThread. The CreateThread function calls a thread function, which is passed some specified information through a struct, which is allocated on the heap. (I find it interesting that even the memory allocation functions in the Windows API are different.)[^fn-7]

### Creating Threads

Windows threads other than the primary thread for each process are created with a call to CreateThread.[^fn-8] In contrast to the 10 parameters necessary for CreateProcess, CreateThread only requires 6 parameters:

{% highlight C %}
for( int i=0; i<MAX_THREADS; i++ ) {
    hThreadArray[i] = CreateThread(NULL, 0,
{% endhighlight %}

The above listing is a typical example of thread creation adopted from code in the Windows API documenta- tion.[^fn-9] In a loop, memory is allocated for the data array, the memory variables are set, and the thread is created and added to the thread array. In this example, the thread function sieve primes is then executed. This is one case where (in my opinion) the Windows API is more clear than the POSIX equivalent, which requires the thread array pointer be passed to pthread create as a variable, rather than returning a reference to the thread.[^fn-10]


### Exiting Threads

`WaitForMultipleObjects(MAX_THREADS, hThreadArray, TRUE, INFINITE);`


## Synchronization

Unlike POSIX, signals to Win32 threads (and ’event’ objects) must be set and reset manually. The benefit of this is that an event need not be looped to persist until a thread reaches it. In Win32 an open event remains open until it is manually closed. If desired, Win32 events may be specified to auto-reset instead.

### WaitForSingleObject

As stated previously, WaitForSingleObject() is the primary method for all HANDLE object synchronization in Windows. It takes the place of waitpid(), pthread join(), and sem wait(). In spite of its flexibility, the call to WaitForSingleObject is very simple, taking only two parameters: a HANDLE and a DWORD timeout value in milliseconds.[^fn-13]

Since Windows processes, threads, mutexes are all HANDLE objects, WaitForSingleObject can wait on all of them. Furthermore, WaitForSingleObject has the added benefit of an optional timeout value, allowing for even more potential uses. Thus, it is entirely up to the programmer to both prevent the misuse of the function as well as naming HANDLE objects in such a way that it is clear when reading code what exactly is being waited for.

The return value is a DWORD representing whether the wait was abandoned, successful, timed out or failed. (WAIT ABANDONED, WAIT OBJECT 0, WAIT TIMEOUT, or WAIT FAILED)

`WaitForMultipleObjects(MAX_THREADS, hThreadArray, TRUE, INFINITE);`

WaitForMultipleObjects is not much more complicated, adding only the number of objects to wait for, an array of HANDLE values, and a boolean bWaitAll (wait for just one of the objects or all of them).[^fn-14] The above example is a typical call from my implementation of the twin prime number finder in Win32. The function waits for MAX THREADS objects contained in the hThreadArray array. The true value referrs to wait for all of the objects, and the timout is set to infinite.

### Mutexes

In Windows, mutexes are initialized by the function CreateMutex (pthreads mutex init in POSIX). A mutex created by one process can be opened by another process with OpenMutex, but only if that mutex has already been initialized.



### Semaphores

Semaphores limit the number of threads which may execute a set of commands simultaneously, based on a specified parameter. Like Win32 mutexes and POSIX named semaphores, Win32 semaphores may be opened by other processes using OpenSemaphore. Beyond that, there seems to be little difference between Win32 semaphores and POSIX semaphores. Remember that system V semaphores are used to synchronize process execution.

### Critical Sections

In addition to the synchronization objects common to both POSIX and Windows, the Windows API contains another distinct object: the CRITICAL SECTION . The function of a Windows Critical Section is similar to the use of a binary pthread mutex to lock and unlock a critical section of code where running potentially non-atomic operations in threads simultaneously might cause incorrect output.




### Events

Another synchronization object unique to the Windows API is the event. Events can be initialized as signaled or not, and can be set to auto-reset or require a manual reset. This is similar to setting a condition variable in POSIX.[^fn-17] Events are ideal for thread communication where threads may be dependent on certain conditions being met in other threads. In addition, events are used by several other pieces of the Win32 API to signal completion.[^fn-18]



Likewise, the APIs for both branches share a common set of tools, though they differ in implementation and interface, they are fundamentally the same tools. Unless you are writing an OS, arguing over which of the two is the ”better” implementation is somewhat moot, as you will always be limited by the platform you are developing for. If you have the time, it is almost always to the programmer’s advantage to write native code rather than emulate a foreign environment for convenience.[^fn-19]

Header image @[flyzipper](https://www.flickr.com/photos/flyzipper/115834464/)
[^fn-2]: It should be noted that using an API wrapper such as [https://www.sourceware.org/pthreads-win32/]() allows programmers to easily port programs from POSIX to the Win32 standard with minimal code changes while still using Win32 underneath. However, using such a wrapper does not offer all of the benefits of the native Windows API.
[^fn-8]: http://msdn.microsoft.com/en-us/library/windows/desktop/ms682453
[^fn-9]: http://msdn.microsoft.com/en-us/library/windows/desktop/ms682516
[^fn-10]: The Windows implementation also requires that a dwThreadIdArray be passed as a parameter, however the usefullness of this array is not as apparent as the array of thread handles.
[^fn-12]: http://msdn.microsoft.com/en-us/library/windows/desktop/ms681917
[^fn-13]: http://msdn.microsoft.com/en-us/library/windows/desktop/ms687032 
[^fn-14]: http://msdn.microsoft.com/en-us/library/windows/desktop/ms687025 
[^fn-15]: http://msdn.microsoft.com/en-us/library/windows/desktop/ms686927
[^fn-17]: http://www.slideshare.net/abufayez/pthreads-vs-win32-threads
[^fn-19]: Attached is the source for Assignment 5’s twin prime number finder using Win32 threads: prime win32.cpp