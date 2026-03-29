# Project 1: Alarm Clock

## Introduction

The objective of this project is to improve the timer mechanism in Pintos kernel. The current implementation of `timer_sleep` in `devices/timer.c` use a 'busy waiting' mechanism, where a sleeping thread checks whether they should wake up or not at every timer tick. Thus, it should be improved by replace the 'busy waiting' design with a new block-based design. Code is available at [https://github.com/NohJanghan/pintos](https://github.com/NohJanghan/pintos.git).

## Problem Definition

The given `timer_sleep` function uses 'busy waiting'. When a thread call `timer_sleep`, it goes to a while statement where it yields the CPU to other threads until the target tick is reached. Since it moves to the back of ready queue after yielding, it will check if it should wake up repeteadly. Thus, it consumes CPU cycles even when the target tick is not reached yet.

As shown in Fig. (TODO) , the 'alarm-multiple` test reports that many kernel ticks was used. If other threads have been running, the tasks would have done late. This problem is worse when many threads are sleeping because each of them checks wether they should wake up at every timer tick. 

## Policy and Algorithm Design

The problem can be solved by avoid using 'busy waiting' approach. Instead of busy waiting, it can be handled using block-based approach. In this approach, the thread which calls `timer_sleep` blocks itself and goes into a sleep queue. The sleep queue is a priority queue which contains threads in sleep state. Then, as timer tick changes, the system checks the sleep queue. If the current timer tick hits or passes by the target ticks which the threads requested, the threads sould be woken up. For this, each thread should maintain a variable storing the tick when it should wake up. The woken thread doesn't hold the CPU immediatley, but it goes into the back of the ready queue and wait for the scheduling.

In this project, the sleep queue was implemented as a sorted list. The threads in the list keep sorted by the target ticks when they will wake up. This implementation assures that target tick checking procedure completes in time complexity of $O(1)$. However, the time complexity of insertion is $O(n)$. The alternative of the sorted list implementation is using a heap. The implementation with heap would be efficient because the time complexities of access and insertion are $O(\log n)$. Nonetheless, the time complexity of $O(n)$ is affordable in a small scale like Pintos. Also, the sorted list provide advantages in terms of simplicity because there already exists the implementation of sorted list in `list.c`.

The entire design can be devided to two algorithms. When a thread enter a sleep state, it puts itself in the sleep list with a variable denoting the target tick and blocks itself. To handle an edge case where a thread wants to sleep for zero tick, a conditional is positioned before the logic. The conditional checks the input and return immediately if the sleep tick is zero.

The other procedure is handled in the timer interrupt. At every timer tick, the timer interrupt checks the first element of the sleep list and if it should be woken up by looking up its target tick. If the thread needs to be woken up, it removed from the list and goes to the ready list, an implementation of the ready queue.

## Implementation

A new static variable `sleep_list` was declared in `timer.c` to manage the sleep queue. The list is an variable of type `struct list` which is implementation of a doubly linked list. The list is initialized with timer interrupt in `timer_init`. When a `thread` goes into `sleep_list`, the function `list_insert_ordered` is called. The function takes a function pointer of type `list_less_func` for sorting. A static function `tick_less` was implemented in `timer.c` to compare two threads' target ticks. Also, a new member variable `tick_to_wakeup` is declared in `thread`, which manage the target tick to wake up.

the function `timer_sleep` is in charge of managing the procedure where a thread goes to sleep. In the function, a thread puts its `elem` in `sleep_list` while setting its `tick_to_wakeup` variable to the sum of current tick and ticks to sleep. After that it blocks itself using the function `thread_block`. The conditional for the edge case checks the input and return immediately if the required sleep tick is zero. Because the function `thread_block` requires to be called with intterupts turned off, `timer_sleep` calles `intr_disable` before blocking and calles `intr_set_level` after unblocking to resume interrupts.

The other procedure is handled in `timer_interrupt`. At every timer tick, `timer_interrupt` checks the first element of the sleep list and if it should be woken up by looking up its `tick_to_wakeup` variable. If the thread needs to be woken up, it pops out of the sleep list and goes to `ready_list`.   Note that the popping out must happen before the insertion to `ready_list`, because the both lists uses a single `list_elem` variable `elem`  in a thread.

## Conclusion

