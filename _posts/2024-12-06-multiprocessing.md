---
title: "The baffling case of Multiprocessing in Python"
date: 2024-12-06
---

On a fateful day, I had to analyze 50GB of application logs. Although structured, the application logs were chaotic at best because the request and response for external calls could be XML or JSON, fields could be missing, etc.

Python works best because I can quickly load the request or response as JSON or XML without breaking a sweat. Furthermore, add multiprocessing, and it becomes fast as well. I sweated for almost 2-3 days before ditching multiprocessing and executing each supposed ChildProcess from the console.

The symptom was that the execution seemed stuck, not moving forward. I allowed the code to execute on my first try because it processes 50 GB of files. I went to sleep, expecting it to be complete by the time I woke up. On further analysis, I realized the execution was stuck. Well, let's get cracking. Below is the minimal code that will get stuck 

```
import multiprocessing as mp

def foo(q):
    with open('shakespear.txt','r') as f:
        for l in f:
            q.put(l)
    print ("Completed q")

def read(q):
    while not q.empty():
        q.get()

if __name__ == '__main__':
    q = mp.Queue()
    p = mp.Process(target=foo,args=(q,))
    p.start()

    p.join()
```

WWhen we execute this script, it won't exit.

My initial assumption was that I would create child processes for each server and read the file simultaneously. While reading, it would serialize the log entries as JSON and put them into a queue. Then, I would read from the queue one at a time and be done. I would go home happy and satisfied.

My first mistake was `p.start()`. I assumed the main Thread would stay at this line until the child processes exit. Looking back, it makes no sense. Why will the execution on the main Thread be stuck at `p.start()`? A child process has been created; it will move ahead.

The second mistake, `p.join()`, will exit once the function returns. This is a big mistake. It doesn't work that way, especially if it is sharing a pipe/queue with a parent process.

To make the program work, I added additional logic to `p.join()` - 
    1. adding a timeout value to the `join()` method . 
    2. checking if the child process has exited (python Process object has exitcode property). 
    3. checking if the queue is fully read.

```
   while True:
        print (f"Process details {p.pid}, {p.is_alive()}, {p.exitcode}")
        print (p.join(2))
        if p.exitcode != None:
            break
        if q.qsize() > 0:
            read(q)
```

I have `procmon` (sysinternal suites) running in background. Take a look at the image below

![PROCESS CREATED](/what-i-learnt/assets/process_created.png)

PID 1692 is the parent process. A few lines below, there is a row for Create Process, and in the details section, PID 2612 is mentioned. This is followed by Process Start for 2612, which is the child process created. The immediate next line is Thread create with thread ID 7348.  

If we check the properties of row `Process Start`, procmon shows the command line to be

> "python.exe" "-c" "from multiprocessing.spawn import spawn_main; spawn_main(parent_pid=1692, pipe_handle=448)" "--multiprocessing-fork"

The main thread in Python is calling module spawn present in multiprocessing folder. The function called is spawn_main.

The code is -

```
def spawn_main(pipe_handle, parent_pid=None, tracker_fd=None):
    '''
    Run code specified by data received over pipe
    '''
    assert is_forking(sys.argv), "Not forking"
    if sys.platform == 'win32':
        import msvcrt
        import _winapi

        if parent_pid is not None:
            source_process = _winapi.OpenProcess(
                _winapi.SYNCHRONIZE | _winapi.PROCESS_DUP_HANDLE,
                False, parent_pid)
        else:
            source_process = None
        new_handle = reduction.duplicate(pipe_handle,
                                         source_process=source_process)
        fd = msvcrt.open_osfhandle(new_handle, os.O_RDONLY)
        parent_sentinel = source_process
```

multiprocessing calls `OpenProcess` with `PROCESS_DUP_HANDLE`, basically creating a bridge between parent process and child process. 

Let's look at the definition of `join` method on python doc

```
If the optional argument timeout is None (the default), the method blocks until the process whose join() method is called terminates. If timeout is a positive number, it blocks at most timeout seconds. Note that the method returns None if its process terminates or if the method times out. Check the process’s exitcode to determine if it terminated.

A process can be joined many times.
```

If `join` is called without timeout(default), it will wait till child process exits.

![PROCESS EXIT](/what-i-learnt/assets/process_exit.png)

There is a row for thread  exit and process exit. Thread `8868` is the main thread and it issues the `ExitProcess`

However, in the `never-ending` version of program, there is no thread exit process for main thread and by extension there is no `ExitProcess`. This is because, the queue is full, has not been read and hence a pipe is still available between child process and parent process, as a result, Child cannot exit.

Refer to the image below, there is process getting created and the main thread. But the main thread never exits, and by extension process doesn't exit. It is waiting for `p.join()` to return, however in this case it is blocking. The console will not even accept `CTRL+C`, we need to kill the console or the parent process from another terminal.

![PROCESS DOESNT EXIT](/what-i-learnt/assets/no_exit.png)



