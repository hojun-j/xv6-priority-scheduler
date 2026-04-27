EECS 3221
Project 2: Priority CPU Scheduling for xv6

Student Name: Hojun Jeon
Student Number: 218877696
Date: 29 March 2026

--------------------------------------------------
1. Overview
--------------------------------------------------

In this project, I modified the xv6 operating system to replace the default
round-robin scheduler with a priority-based scheduler and an aging mechanism.

The scheduler now selects the highest-priority RUNNABLE process instead of
simply running processes in round-robin order. To prevent starvation of
low-priority processes, I also implemented aging so that a process waiting
for CPU time can gradually gain priority.

This project was built on top of my Project 1 xv6 source, which already
included the getprocs system call and the ps user program.

--------------------------------------------------
2. Features Implemented
--------------------------------------------------

The following features were implemented for this project:

1. Priority-based scheduling
   - Each process has a priority value.
   - Priority range is from 0 to 10.
   - Higher number means higher priority.
   - Default priority is 5.

2. Priority inheritance
   - When a process forks, the child inherits the parent’s priority settings.

3. Aging mechanism
   - Each process stores both a base priority and a wait time.
   - While a process stays RUNNABLE and is not selected to run, its wait time
     increases.
   - When the wait time reaches the aging threshold, the process priority is
     increased, up to the maximum priority.
   - When a process is selected to run, its current priority is reset to its
     base priority and its wait time is reset.

4. System calls
   - setpriority(int pid, int priority)
   - getpriority(int pid)

5. Testing support
   - A user program named priority_test was added to demonstrate priority
     scheduling and aging behavior.

--------------------------------------------------
3. Files Modified
--------------------------------------------------

The following files were modified as part of this project:

kernel/proc.h
- Added scheduling-related fields to struct proc:
  - priority
  - base_priority
  - wait_time

kernel/proc.c
- Initialized default scheduling values in allocproc().
- Modified kfork() so that child processes inherit the parent’s priority.
- Replaced the original scheduler() logic with priority-based selection.
- Added aging logic in scheduler().
- Added kernel helper functions for priority management.

kernel/syscall.h
- Added system call numbers for setpriority and getpriority.

kernel/syscall.c
- Registered the new system calls in the system call table.

kernel/sysproc.c
- Added system call handlers:
  - sys_setpriority()
  - sys_getpriority()
- Also kept my previous Project 1 support code for getprocs/ps.

kernel/defs.h
- Added function declarations for the new kernel priority functions.

user/user.h
- Added user-level function declarations for setpriority() and getpriority().

user/usys.pl
- Added user stubs for the new system calls.

user/priority_test.c
- Added a test program to create multiple child processes with different
  priorities and demonstrate scheduler behavior.

Makefile
- Added priority_test to the user programs.
- For a cleaner demonstration, I set the QEMU CPU count to 1 so that console
  output is easier to read during testing.

--------------------------------------------------
4. Design and Implementation Details
--------------------------------------------------

A. Priority Fields

Each process contains:
- priority: the current effective priority used by the scheduler
- base_priority: the original priority that should be restored after running
- wait_time: the amount of time the process has been waiting while RUNNABLE

B. Scheduler Logic

The scheduler scans the process table and selects the RUNNABLE process with
the highest priority.

If two processes have the same priority, the process with the smaller PID is
chosen as a tie-breaker.

C. Aging Logic

To prevent starvation, every RUNNABLE process that is not currently running
has its wait_time incremented while the scheduler scans processes.

If wait_time reaches the aging threshold, the process priority is increased
by 1, up to the maximum allowed priority of 10.

When the selected process is about to run:
- its priority is reset to base_priority
- its wait_time is reset to 0

I also added a debug print in the scheduler to show when aging occurs during
testing. This makes it easier to demonstrate that low-priority processes gain
priority while waiting.

D. System Call Behavior

setpriority(pid, priority)
- Returns success when the target process exists and the priority is valid.
- Rejects invalid priority values outside the range 0 to 10.
- Updates both priority and base_priority for the target process.
- Resets wait_time to 0.

getpriority(pid)
- Returns the current priority of the target process.
- Returns -1 if the process is not found.

--------------------------------------------------
5. Concurrency and Safety
--------------------------------------------------

Since this project modifies kernel scheduling behavior, I paid attention to
safe lock handling while accessing process state.

For process-level scheduling data such as state, priority, and wait_time,
the process lock is used before reading or writing those fields.

In addition, because this project builds on my Project 1 code, I also fixed
the locking issues in my previous getprocs implementation:
- copyout is no longer performed while holding a process spinlock
- parent process information is accessed safely with the appropriate wait lock

These fixes were important to avoid race conditions and kernel instability.

--------------------------------------------------
6. Testing
--------------------------------------------------

I tested the implementation using the priority_test user program.

Test setup:
- Three child processes are created.
- Their base priorities are set to:
  - high = 8
  - medium = 5
  - low = 2

Expected behavior:
- The high-priority process should run first.
- The medium-priority process should run before the low-priority one.
- The low-priority process should still eventually run because of aging.
- All child processes should complete successfully.

Observed behavior:
- The high-priority child completed first.
- The medium-priority child completed next.
- The low-priority child eventually completed as well.
- The aging debug output showed that waiting processes had their priorities
  increased over time.
- The final message confirmed that all children completed successfully.

I also verified that the ps command from Project 1 still works correctly
after the Project 2 scheduler changes.

--------------------------------------------------
7. How to Run
--------------------------------------------------

1. Enter the xv6 project directory.
2. Run:
   make clean
   make qemu

3. In the xv6 shell, run:
   ps
   priority_test

The ps command shows process information from Project 1.
The priority_test command demonstrates priority scheduling and aging.

--------------------------------------------------
8. Summary
--------------------------------------------------

This project successfully replaces xv6’s default round-robin scheduler with a
priority-based scheduler and adds an aging mechanism to prevent starvation.

The implementation supports:
- priority assignment
- priority inheritance on fork
- dynamic aging for waiting processes
- user-level priority system calls
- a demonstration program showing scheduler behavior

The final system builds and runs correctly in xv6, and the test program shows
that higher-priority processes get preference while lower-priority processes
still eventually receive CPU time.
