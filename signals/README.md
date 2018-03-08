## Overview  

Sending a signal means updating signals queue for the thread. Only one copy is maintained. Also, signal function’s name is  Signal as signal() already was another function’s name.  Updated Makefile.build in src directory.

## Files Added / Modified

### Thread.h/c (Mainly explains Data Structures used)
1. Added global to_unblock_list for threads to be unblocked in next context switch
2. Added tids hash table for tid -> thread *
3. Added to struct thread:
    1. long long lifetime: lifetime of thread
    2. long long ticks: time spent by thread
    3. Struct hash_elem: for hash table to convert tid -> thread * ➢ Int ptid: parent’s tid
    4. Sigset_t mask: bitmask for what signals are blocked(on) ➢ Signals_queue: queue for signals.(at most 4)
4. Added thread_lookup(int) for finding thread * from tid using hash table. Also added tid_hash, tid_less required by hash table (inbuilt in pintos)
5. Thread_tick: For running/ready threads update ticks, if it exceeds their lifetime, provide a SIG_CPU to them(if not blocked/ignored)
6. Thread_create: init data structures
7. Thread_exit: signal SIG_CHLD to parent if not blocked/ignored.
8. Init_thread: initialized struct thread parameters
9. Schedule: at end, unblock all threads in to_unblock_list if they are blocked. Also for the next thread, process all signals. Call their handlers.
10. Setlifetime: Update thread’s lifetime

### Signal.h/c
1. Added #define for different constants like SIG_CHLD, SIG_BLOCK etc.
2. Enum sighandler_t for SIG_DFL or SIG_IGN
3. Struct signal_t
    1. Int type
    2. Int sent_by: tid of last thread which sent this signal
4. Typedef unsigned short sigset_t: bitmask for 4 signals
5. Functions are made atomic by disabling interrupts and reenabling them.
6. Signal: Ignore if SIG_KILL. Check old handler using mask; if not same, flip the bit.
7. Kill: Find thread * using hash table. Ignore if SIG_CHLD or SIG_CPU or it is for OS thread(1 or 2). If SIG_UBLOCK, add to global unblock queue. If SIG_KILL check if valid by checking ptid. Push signal in thread’s queue if not exists. If it exists, then only update the sent_by field. Also initial check was done if this signal was blocked. 
8. Sigprocmask: if oldset is not NULL, update it to current mask. If set is NULL, return. Use bitwise operations to update masks depending on the how parameter. If how parameter is unknown, return -1.
9. Sigempty, fill, add, del: If set is NULL, return -1. Else, update it using corresponding bitwise operations.
10. Default handlers: SIG_KILL_DFL, SIG_USER_DFL, SIG_CPU_DFL, SIG_CHLD_DFL, Printed stuff as given in assignment. KILL, CPU ones call thread_exit
