# xv6-priority-scheduler

Modified the xv6 operating system to replace the default round-robin scheduler with a priority-based CPU scheduler and aging mechanism.

## Features

- Implemented priority-based scheduling
- Added aging to prevent starvation
- Added `setpriority()` and `getpriority()` system calls
- Added `priority_test` user program for testing
- Built on top of Project 1 (`getprocs()` and `ps`)

## Tech Stack

- C
- xv6 Operating System
- QEMU

## Usage

```bash
make clean
make qemu

Run inside xv6 shell:
ps
priority_test

Author
Hojun Jeon
